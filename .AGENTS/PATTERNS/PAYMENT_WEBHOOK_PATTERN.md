# 💳 PATTERN — Penanganan Webhook Pembayaran Midtrans yang Idempoten & Terverifikasi (PAYMENT_WEBHOOK_PATTERN)

- **Applies To:** Backend / Webhooks / Integrasi Pembayaran
- **Framework:** Next.js 15 Route Handlers, Node.js Crypto (Native), Prisma ORM, Resend
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Menangani notifikasi status pembayaran asinkron server-ke-server dari Midtrans Snap di endpoint `POST /api/payments/midtrans/webhook`.
- Memverifikasi keabsahan data masuk menggunakan kalkulasi tanda tangan kriptografi SHA-512 dengan `MIDTRANS_SERVER_KEY`.
- Menegakkan sifat **idempoten mutlak** (*absolute idempotency*) sehingga pemanggilan webhook berulang untuk pesanan yang sama tidak memicu mutasi status ganda atau pengiriman email konfirmasi duplikat.
- Mengubah status entitas `payments` dan `orders` secara atomik dalam satu transaksi basis data serta memicu pengiriman email faktur resmi.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Pembayaran transfer bank manual — gunakan Server Action `approveManualPaymentAction` di panel admin.
- Redirect sukses sisi klien peramban (*client finish redirect*) — jangan pernah mengubah status pembayaran menjadi lunas hanya berdasarkan callback peramban pengguna; selalu tunggu webhook server-ke-server resmi.

---

## 3. Struktur Blueprint (Structure)

```
src/
├── app/
│   └── api/
│       └── payments/
│           └── midtrans/
│               └── webhook/
│                   └── route.ts       # Route Handler penerima webhook Midtrans
└── lib/
    ├── payments/
    │   └── midtrans.ts                # Utilitas verifikasi signature SHA-512
    ├── email/
    │   └── send-order-confirmation.ts # Pemicu email faktur Resend
    └── db.ts                          # Prisma Client singleton
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah implementasi lengkap fungsi verifikasi kriptografi dan handler rute webhook di `src/app/api/payments/midtrans/webhook/route.ts`:

### 1. Modul Verifikasi Tanda Tangan SHA-512 (`src/lib/payments/midtrans.ts`)

```typescript
import crypto from 'node:crypto';

export interface MidtransNotificationPayload {
  transaction_time: string;
  transaction_status: string;
  transaction_id: string;
  status_message: string;
  status_code: string;
  signature_key: string;
  payment_type: string;
  order_id: string;
  gross_amount: string;
  fraud_status?: string;
  currency?: string;
  settlement_time?: string;
}

/**
 * Memverifikasi integritas payload Midtrans menggunakan algoritma SHA-512.
 * Formula: SHA512(order_id + status_code + gross_amount + ServerKey)
 */
export function verifyMidtransSignature(
  payload: MidtransNotificationPayload,
  serverKey: string
): boolean {
  const { order_id, status_code, gross_amount, signature_key } = payload;

  if (!order_id || !status_code || !gross_amount || !signature_key || !serverKey) {
    return false;
  }

  const rawString = `${order_id}${status_code}${gross_amount}${serverKey}`;
  const computedHash = crypto
    .createHash('sha512')
    .update(rawString)
    .digest('hex');

  // Gunakan timingSafeEqual untuk mencegah serangan side-channel timing attack
  const computedBuffer = Buffer.from(computedHash, 'utf-8');
  const signatureBuffer = Buffer.from(signature_key, 'utf-8');

  if (computedBuffer.length !== signatureBuffer.length) {
    return false;
  }

  return crypto.timingSafeEqual(computedBuffer, signatureBuffer);
}
```

### 2. Route Handler Webhook Idempoten (`src/app/api/payments/midtrans/webhook/route.ts`)

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/db';
import { verifyMidtransSignature, MidtransNotificationPayload } from '@/lib/payments/midtrans';
import { sendOrderConfirmationEmail } from '@/lib/email/send-order-confirmation';

export async function POST(request: NextRequest) {
  try {
    const payload: MidtransNotificationPayload = await request.json();

    const serverKey = process.env.MIDTRANS_SERVER_KEY;
    if (!serverKey) {
      console.error('[Midtrans Webhook Error]: MIDTRANS_SERVER_KEY belum dikonfigurasi di environment variables.');
      return NextResponse.json({ message: 'Server configuration error' }, { status: 500 });
    }

    // 1. Verifikasi Tanda Tangan SHA-512
    const isValidSignature = verifyMidtransSignature(payload, serverKey);
    if (!isValidSignature) {
      console.warn(`[Midtrans Webhook Security]: Tanda tangan tidak valid untuk order_id: ${payload.order_id}`);
      return NextResponse.json({ message: 'Invalid signature key' }, { status: 401 });
    }

    const { order_id, transaction_status, fraud_status, transaction_id } = payload;

    // 2. Cari Pesanan Terkait di Basis Data
    const order = await prisma.order.findUnique({
      where: { orderNumber: order_id },
      include: {
        payments: {
          orderBy: { createdAt: 'desc' },
          take: 1,
        },
      },
    });

    if (!order) {
      console.warn(`[Midtrans Webhook]: Pesanan tidak ditemukan: ${order_id}`);
      return NextResponse.json({ message: 'Order not found' }, { status: 404 });
    }

    // 3. Pemeriksaan Idempotensi (Idempotency Check)
    // Jika pesanan sudah berstatus 'paid', kembalikan 200 OK langsung tanpa memicu mutasi ulang
    if (order.status === 'paid') {
      return NextResponse.json({ message: 'Notification already processed (idempotent)' }, { status: 200 });
    }

    const latestPayment = order.payments[0];

    // 4. Evaluasi Transisi Status Pembayaran Midtrans
    let isPaymentSuccess = false;
    let isPaymentFailed = false;

    if (transaction_status === 'capture') {
      // Khusus transaksi kartu kredit: cek status penipuan (fraud_status)
      if (fraud_status === 'accept') {
        isPaymentSuccess = true;
      } else if (fraud_status === 'challenge') {
        // Status challenge ditinjau manual oleh merchant
        console.info(`[Midtrans]: Transaksi ${order_id} memerlukan peninjauan fraud manual.`);
      }
    } else if (transaction_status === 'settlement') {
      // Pembayaran selesai (VA, QRIS, Gopay, transfer bank)
      isPaymentSuccess = true;
    } else if (transaction_status === 'cancel' || transaction_status === 'deny' || transaction_status === 'expire') {
      isPaymentFailed = true;
    }

    // 5. Eksekusi Pembaruan Transaksional Atomik
    if (isPaymentSuccess) {
      await prisma.$transaction(async (tx) => {
        // Perbarui status record pembayaran
        if (latestPayment) {
          await tx.payment.update({
            where: { id: latestPayment.id },
            data: {
              status: 'paid',
              providerRef: transaction_id,
            },
          });
        }

        // Perbarui status order menjadi 'paid'
        await tx.order.update({
          where: { id: order.id },
          data: {
            status: 'paid',
          },
        });
      });

      // 6. Memicu Email Konfirmasi Pembayaran Resmi secara Asinkron
      const targetEmail = order.guestEmail ?? (await getCustomerEmail(order.userId));
      if (targetEmail) {
        // Eksekusi pengiriman email faktur via Resend tanpa memblokir respon HTTP
        sendOrderConfirmationEmail({
          orderNumber: order.orderNumber,
          recipientEmail: targetEmail,
          totalMinor: order.totalMinor,
          currency: order.currency,
        }).catch((err) => console.error('[Email Notification Error]:', err));
      }
    } else if (isPaymentFailed) {
      await prisma.$transaction(async (tx) => {
        if (latestPayment) {
          await tx.payment.update({
            where: { id: latestPayment.id },
            data: {
              status: transaction_status === 'expire' ? 'expired' : 'failed',
              providerRef: transaction_id,
            },
          });
        }

        await tx.order.update({
          where: { id: order.id },
          data: {
            status: 'cancelled',
          },
        });
      });
    }

    return NextResponse.json({ message: 'Webhook processed successfully' }, { status: 200 });
  } catch (error: unknown) {
    console.error('[Midtrans Webhook Fatal Error]:', error);
    return NextResponse.json({ message: 'Internal server error processing webhook' }, { status: 500 });
  }
}

async function getCustomerEmail(userId: string | null): Promise<string | null> {
  if (!userId) return null;
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { email: true },
  });
  return user?.email ?? null;
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

```typescript
// ❌ ANTI-PATTERN 1: Memproses webhook tanpa memvalidasi signature hash
export async function badWebhook(req: Request) {
  const data = await req.json();
  // BENCANA INTEGRITAS FINANSIAL: Siapa pun di internet bisa memalsukan status pembayaran menjadi sukses!
  await prisma.order.update({ where: { orderNumber: data.order_id }, data: { status: 'paid' } });
  return NextResponse.json({ ok: true });
}

// ❌ ANTI-PATTERN 2: Menggunakan perbandingan string standar (==) untuk hash
if (computedHash === signature_key) {
  // Rentan terhadap timing attack pada skala presisi mikrosekon; selalu gunakan crypto.timingSafeEqual
}

// ❌ ANTI-PATTERN 3: Mengirim email konfirmasi berulang kali tanpa idempotency check
// Jika Midtrans mengirim 3 webhook retry berturut-turut, pelanggan akan menerima 3 email faktur yang sama
```

---

## 6. Aturan & Invariants (Rules)

1. **Verifikasi Hash Wajib & Mutlak:** Tidak ada satu pun baris data pesanan atau pembayaran yang boleh diperbarui sebelum hash SHA-512 diverifikasi dan lolos pencocokan aman `timingSafeEqual`.
2. **Kepatuhan Idempotensi:** Jika pesanan telah berstatus `paid`, server harus mengembalikan status `200 OK` tanpa memicu ulang transaksi atau notifikasi.
3. **Pembaruan Status Atomik:** Perubahan status entitas `payments` dan `orders` wajib dibungkus dalam blok `prisma.$transaction()`.
4. **Penanganan Asinkron Notifikasi Email:** Pengiriman email konfirmasi (Resend) tidak boleh menghambat (*unblock*) pengembalian respon HTTP 200 OK ke Midtrans.
5. **Pencatatan Log Audit Transaksi:** Catat selalu `transaction_id` penyedia gerbang pembayaran ke kolom `providerRef` sebagai bukti rekonsiliasi keuangan.

---

## 7. Pola & Dokumen Terkait

- [PAYMENT.md](../FEATURES/PAYMENT.md) — Spesifikasi alur pembayaran Midtrans
- [API_RESPONSE_PATTERN.md](./API_RESPONSE_PATTERN.md) — Pola respon API Route Handler
- [ORDERS.md](../FEATURES/ORDERS.md) — Siklus hidup status pesanan
