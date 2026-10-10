# 📬 PATTERN — Standarisasi Envelope Respon JSON API (API_RESPONSE_PATTERN)

- **Applies To:** Backend / Route Handlers (`src/app/api/**/route.ts`)
- **Framework:** Next.js 15 (NextResponse), TypeScript (Strict)
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Seluruh endpoint API Route Handler di bawah direktori `src/app/api/**/route.ts` (misal: webhook Midtrans, pencarian publik berlian, slot janji temu, atau kueri katalog eksternal).
- Menghasilkan format amplop (*envelope*) respon JSON yang seragam di seluruh aplikasi, baik untuk skenario sukses (`success: true`) maupun skenario kegagalan (`success: false`).
- Menyelaraskan kode status HTTP (200, 201, 400, 401, 403, 404, 409, 429, 500) dengan payload respon yang terstruktur rapi.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Next.js Server Actions — Server Actions mengembalikan objek kembalian langsung atau payload terpisah, bukan instance `NextResponse`.
- Streaming respon Server-Sent Events (SSE) atau pengunduhan file biner langsung (PDF faktur / berkas .ics) — gunakan `Response` native stream.

---

## 3. Struktur Blueprint (Structure)

```
src/
└── lib/
    └── api/
        ├── response.ts        # Helper apiSuccess() & apiError() serta tipe envelope
        └── errors.ts          # Definisi kode error standar aplikasi
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah modul utilitas `src/lib/api/response.ts` lengkap beserta contoh konsumsinya dalam Route Handler `src/app/api/appointments/slots/route.ts`:

### 1. Helper Envelope Standar (`src/lib/api/response.ts`)

```typescript
import { NextResponse } from 'next/server';

/**
 * Format amplop respon sukses
 */
export interface ApiSuccessEnvelope<T> {
  success: true;
  data: T;
  meta?: {
    page?: number;
    limit?: number;
    total?: number;
    [key: string]: unknown;
  };
}

/**
 * Format amplop respon error
 */
export interface ApiErrorEnvelope {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, string[] | string> | null;
  };
}

export type ApiResponseEnvelope<T> = ApiSuccessEnvelope<T> | ApiErrorEnvelope;

/**
 * Membentuk respon JSON sukses yang terstandarisasi.
 *
 * @param data Payload data yang dikembalikan
 * @param status Kode status HTTP (default: 200 OK)
 * @param meta Metadata opsional (misal: penomoran halaman atau total hasil)
 */
export function apiSuccess<T>(
  data: T,
  status: number = 200,
  meta?: ApiSuccessEnvelope<T>['meta']
): NextResponse<ApiSuccessEnvelope<T>> {
  return NextResponse.json(
    {
      success: true,
      data,
      ...(meta ? { meta } : {}),
    },
    { status }
  );
}

/**
 * Membentuk respon JSON error yang terstandarisasi.
 *
 * @param code Kode error mesin (misal: 'VALIDATION_ERROR', 'SLOT_UNAVAILABLE')
 * @param message Pesan error ramah pengguna dalam Bahasa Indonesia
 * @param status Kode status HTTP (default: 400 Bad Request)
 * @param details Rincian validasi field Zod atau error pelengkap
 */
export function apiError(
  code: string,
  message: string,
  status: number = 400,
  details?: Record<string, string[] | string> | null
): NextResponse<ApiErrorEnvelope> {
  return NextResponse.json(
    {
      success: false,
      error: {
        code,
        message,
        details: details ?? null,
      },
    },
    { status }
  );
}
```

### 2. Contoh Penggunaan dalam Route Handler (`src/app/api/appointments/slots/route.ts`)

```typescript
import { NextRequest } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/db';
import { apiSuccess, apiError } from '@/lib/api/response';

const QuerySchema = z.object({
  boutiqueId: z.string().uuid('ID Butik tidak valid'),
  date: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, 'Format tanggal harus YYYY-MM-DD'),
});

const DEFAULT_HOURLY_SLOTS = ['11:00', '13:00', '14:00', '15:00', '16:00', '17:00', '18:00', '19:00'];

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const parsed = QuerySchema.safeParse({
      boutiqueId: searchParams.get('boutiqueId'),
      date: searchParams.get('date'),
    });

    if (!parsed.success) {
      return apiError(
        'VALIDATION_ERROR',
        'Parameter pencarian jadwal janji temu tidak valid.',
        400,
        parsed.error.flatten().fieldErrors
      );
    }

    const { boutiqueId, date } = parsed.data;

    // Pastikan butik aktif
    const boutique = await prisma.boutique.findUnique({
      where: { id: boutiqueId, isActive: true },
    });

    if (!boutique) {
      return apiError(
        'BOUTIQUE_NOT_FOUND',
        'Butik fisik yang dipilih tidak ditemukan atau sedang tidak beroperasi.',
        404
      );
    }

    // Ambil seluruh janji temu terkonfirmasi pada tanggal tersebut
    const startOfDay = new Date(`${date}T00:00:00+07:00`);
    const endOfDay = new Date(`${date}T23:59:59+07:00`);

    const existingAppointments = await prisma.appointment.findMany({
      where: {
        boutiqueId,
        scheduledAt: { gte: startOfDay, lte: endOfDay },
        status: { in: ['requested', 'confirmed'] },
      },
      select: { scheduledAt: true },
    });

    // Petakan slot terisi
    const bookedHours = new Set(
      existingAppointments.map((apt) => {
        const d = new Date(apt.scheduledAt);
        return `${String(d.getHours()).padStart(2, '0')}:00`;
      })
    );

    const slots = DEFAULT_HOURLY_SLOTS.map((time) => ({
      time,
      available: !bookedHours.has(time),
    }));

    return apiSuccess(
      {
        boutiqueId,
        boutiqueName: boutique.name,
        date,
        slots,
      },
      200
    );
  } catch (error: unknown) {
    console.error('[API Slots Error]:', error);
    return apiError(
      'INTERNAL_SERVER_ERROR',
      'Terjadi gangguan saat mengambil slot janji temu.',
      500
    );
  }
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

```typescript
// ❌ ANTI-PATTERN 1: Format respon acak dan tidak terbungkus amplop standar
export async function badHandler() {
  // Mengembalikan array mentah tanpa indikator status eksplisit
  return NextResponse.json([{ id: 1, name: 'Piece' }]);
}

// ❌ ANTI-PATTERN 2: Mengembalikan status 200 OK saat terjadi error
export async function badErrorHandler() {
  // Sangat membingungkan klien karena status HTTP 200 padahal terjadi error validasi!
  return NextResponse.json({ success: false, error: 'Data invalid' }, { status: 200 });
}

// ❌ ANTI-PATTERN 3: Melempar error mentah basis data tanpa enkapsulasi
export async function badCatchHandler(error: any) {
  // Membocorkan kredensial atau struktur query internal ke publik
  return NextResponse.json({ message: error.stack }, { status: 500 });
}
```

---

## 6. Aturan & Invariants (Rules)

1. **Seluruh Respon Terbungkus Envelope:** Semua route handler wajib menggunakan `apiSuccess()` atau `apiError()` tanpa terkecuali.
2. **Kesesuaian Kode Status HTTP:**
   - `200 OK`: Sukses pengambilan data atau pemrosesan umum.
   - `201 Created`: Sukses pembuatan data baru (pesanan, janji temu).
   - `400 Bad Request`: Validasi skema Zod gagal atau parameter tidak lengkap.
   - `401 Unauthorized`: Autentikasi hilang atau tanda tangan webhook invalid.
   - `403 Forbidden`: Sesi tidak memiliki wewenang peran yang mencukupi.
   - `404 Not Found`: Entitas tidak ditemukan di basis data.
   - `409 Conflict`: Bentrokan slot waktu janji temu atau inventaris habis.
   - `429 Too Many Requests`: Melebihi batas kuota pemanggilan (*rate limit*).
   - `500 Internal Server Error`: Kegagalan tak terduga pada server.
3. **Pesan Bahasa Indonesia yang Ramah:** Pesan error pada properti `error.message` wajib ditulis dalam Bahasa Indonesia yang santun dan jelas bagi pengguna akhir.
4. **Kode Mesin Huruf Besar (UPPER_SNAKE_CASE):** Properti `error.code` wajib berupa string konstan yang mudah dievaluasi oleh klien (misal: `VALIDATION_ERROR`, `SLOT_UNAVAILABLE`, `INVALID_SIGNATURE`).

---

## 7. Pola & Dokumen Terkait

- [API_CONTRACT.md](../API_CONTRACT.md) — Dokumen kontrak API sistem
- [PAYMENT_WEBHOOK_PATTERN.md](./PAYMENT_WEBHOOK_PATTERN.md) — Implementasi respon pada webhook
