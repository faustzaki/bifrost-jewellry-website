# ⚡ PATTERN — Server Action dengan Validasi Zod & RBAC (SERVER_ACTION_PATTERN)

- **Applies To:** Backend / Mutations / Next.js Server Actions
- **Framework:** Next.js 15 (App Router), Prisma ORM, Zod, Auth.js v5
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Seluruh operasi mutasi data (Create, Update, Delete) yang dipicu oleh interaksi pengguna di peramban, baik di sisi publik (formulir kontak, booking janji temu, penambahan item ke keranjang) maupun di area admin (`/admin`).
- Operasi mutasi internal panel admin (menyetujui bukti transfer manual, mengubah status pesanan, memperbarui katalog perhiasan dan spesifikasi berlian).
- Operasi yang memerlukan revalidasi cache Next.js ISR secara on-demand menggunakan `revalidatePath` atau `revalidateTag`.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Operasi pembacaan data murni (*read-only fetching*) — gunakan langsung React Server Component (RSC) dengan query Prisma asinkron di dalam komponen `page.tsx`.
- Handler webhook pihak ketiga (seperti Midtrans Payment Webhook) — gunakan Route Handler HTTP POST di `src/app/api/**/route.ts` karena webhook memerlukan kontrol status code HTTP dan parsing raw payload eksternal.
- Endpoint publik yang dikonsumsi oleh aplikasi klien pihak ketiga eksternal.

---

## 3. Struktur Blueprint (Structure)

```
src/
├── app/
│   └── admin/
│       └── pieces/
│           ├── actions.ts           # Server Actions untuk manajemen karya perhiasan
│           ├── schemas.ts           # Skema validasi Zod input
│           └── page.tsx             # Halaman RSC penampil data
└── lib/
    ├── auth.ts                      # Helper sesi Auth.js (auth())
    ├── db.ts                        # Inisialisasi Prisma Client singleton
    └── errors.ts                    # Kelas error domain terstandarisasi
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah implementasi Server Action nyata untuk membuat karya perhiasan baru beserta spesifikasi batu berlian dalam satu transaksi atomik Prisma, lengkap dengan verifikasi peran admin dan revalidasi cache publik:

```typescript
// src/app/admin/pieces/schemas.ts
import { z } from 'zod';

export const DiamondOriginEnum = z.enum(['natural', 'lab_grown']);
export const DiamondCutEnum = z.enum(['excellent', 'very_good', 'good', 'fair']);
export const DiamondShapeEnum = z.enum(['round', 'oval', 'emerald', 'pear', 'princess', 'marquise', 'cushion']);
export const GradingLabEnum = z.enum(['GIA', 'IGI']);
export const MetalEnum = z.enum(['gold_18k', 'gold_14k', 'platinum', 'rose_gold_18k']);
export const AvailabilityEnum = z.enum(['in_stock', 'made_to_order', 'reserved', 'sold_out']);

export const CreatePieceSchema = z.object({
  name: z.string().trim().min(3, 'Nama perhiasan minimal 3 karakter').max(120),
  slug: z.string().trim().toLowerCase().regex(/^[a-z0-9-]+$/, 'Slug hanya boleh berisi huruf kecil, angka, dan strip'),
  sku: z.string().trim().toUpperCase().min(3, 'SKU minimal 3 karakter'),
  description: z.string().trim().min(10, 'Deskripsi perhiasan minimal 10 karakter'),
  categoryId: z.string().uuid('ID Kategori tidak valid'),
  collectionId: z.string().uuid('ID Koleksi tidak valid').optional().nullable(),
  metal: MetalEnum,
  settingType: z.string().trim().min(2, 'Jenis tatahan wajib diisi'),
  priceIdrMinor: z.number().int().positive('Harga IDR harus berupa angka positif dalam satuan minor'),
  priceUsdMinor: z.number().int().positive('Harga USD harus berupa angka positif dalam satuan minor'),
  availability: AvailabilityEnum.default('in_stock'),
  isPublished: z.boolean().default(false),
  isFeatured: z.boolean().default(false),
  // Spesifikasi berlian terlampir (opsional)
  diamond: z.object({
    origin: DiamondOriginEnum,
    carat: z.number().positive().max(50, 'Karat tidak boleh melebihi 50 ct'),
    cut: DiamondCutEnum,
    colorGrade: z.string().trim().toUpperCase().min(1).max(3),
    clarityGrade: z.string().trim().toUpperCase().min(1).max(5),
    shape: DiamondShapeEnum,
    gradingLab: GradingLabEnum,
    certificateNumber: z.string().trim().min(4, 'Nomor sertifikat minimal 4 karakter'),
    certificateUrl: z.string().url('URL sertifikat tidak valid').optional().nullable(),
  }).optional().nullable(),
});

export type CreatePieceInput = z.infer<typeof CreatePieceSchema>;
```

```typescript
// src/app/admin/pieces/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { prisma } from '@/lib/db';
import { auth } from '@/lib/auth';
import { CreatePieceSchema } from './schemas';

export type ActionResponse<T> =
  | { success: true; data: T }
  | { success: false; error: { code: string; message: string; details?: Record<string, string[]> } };

/**
 * Server Action untuk membuat karya perhiasan baru beserta batu berlian bersertifikat.
 * Dilindungi hak akses admin/super_admin dan dieksekusi secara atomik.
 */
export async function createPieceAction(rawInput: unknown): Promise<ActionResponse<{ pieceId: string; slug: string }>> {
  try {
    // 1. Verifikasi Sesi & Hak Akses (RBAC) di sisi server
    const session = await auth();
    if (!session?.user || !['admin', 'super_admin'].includes(session.user.role)) {
      return {
        success: false,
        error: {
          code: 'UNAUTHORIZED',
          message: 'Akses ditolak: Anda tidak memiliki wewenang administratif untuk melakukan tindakan ini.',
        },
      };
    }

    // 2. Validasi Input ketat dengan Zod
    const validationResult = CreatePieceSchema.safeParse(rawInput);
    if (!validationResult.success) {
      return {
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Input formulir karya perhiasan tidak valid.',
          details: validationResult.error.flatten().fieldErrors,
        },
      };
    }

    const input = validationResult.data;

    // 3. Eksekusi Transaksi Atomik Prisma
    const createdPiece = await prisma.$transaction(async (tx) => {
      // Pastikan SKU dan slug belum digunakan
      const existingSku = await tx.piece.findFirst({
        where: { OR: [{ sku: input.sku }, { slug: input.slug }] },
      });
      if (existingSku) {
        throw new Error('DUPLICATE_IDENTIFIER: SKU atau Slug perhiasan tersebut sudah terdaftar di sistem.');
      }

      // Buat entitas piece utama
      const piece = await tx.piece.create({
        data: {
          name: input.name,
          slug: input.slug,
          sku: input.sku,
          description: input.description,
          categoryId: input.categoryId,
          collectionId: input.collectionId ?? null,
          metal: input.metal,
          settingType: input.settingType,
          priceIdrMinor: input.priceIdrMinor,
          priceUsdMinor: input.priceUsdMinor,
          availability: input.availability,
          isPublished: input.isPublished,
          isFeatured: input.isFeatured,
        },
      });

      // Buat spesifikasi berlian jika disertakan
      if (input.diamond) {
        await tx.diamond.create({
          data: {
            pieceId: piece.id,
            origin: input.diamond.origin,
            carat: input.diamond.carat,
            cut: input.diamond.cut,
            colorGrade: input.diamond.colorGrade,
            clarityGrade: input.diamond.clarityGrade,
            shape: input.diamond.shape,
            gradingLab: input.diamond.gradingLab,
            certificateNumber: input.diamond.certificateNumber,
            certificateUrl: input.diamond.certificateUrl ?? null,
          },
        });
      }

      return piece;
    });

    // 4. On-demand Cache Revalidation untuk halaman publik terkait
    revalidatePath('/collections');
    revalidatePath(`/collections/${input.categoryId}`);
    revalidatePath('/diamonds');
    revalidatePath('/admin/pieces');

    // 5. Kembalikan Respon Terstruktur Sukses
    return {
      success: true,
      data: {
        pieceId: createdPiece.id,
        slug: createdPiece.slug,
      },
    };
  } catch (error: unknown) {
    // Tangani error domain atau Prisma
    const errorMessage = error instanceof Error ? error.message : 'Terjadi kegagalan sistem internal saat menyimpan karya.';
    const isDuplicate = errorMessage.startsWith('DUPLICATE_IDENTIFIER:');

    return {
      success: false,
      error: {
        code: isDuplicate ? 'CONFLICT' : 'INTERNAL_SERVER_ERROR',
        message: isDuplicate ? errorMessage.replace('DUPLICATE_IDENTIFIER: ', '') : 'Gagal menyimpan karya perhiasan.',
      },
    };
  }
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

Hindari pola-pola salah berikut:

```typescript
// ❌ ANTI-PATTERN 1: Melewatkan verifikasi sesi dan validasi tipe (menggunakan 'any')
export async function badCreatePieceAction(data: any) {
  'use server';
  // Bahaya: Pengguna umum bisa memicu aksi ini tanpa autentikasi!
  // Bahaya: Tidak ada validasi skema Zod
  return await prisma.piece.create({ data });
}

// ❌ ANTI-PATTERN 2: Mutasi multi-tabel tanpa transaksi atomik
export async function badMultiWrite(data: any) {
  'use server';
  const piece = await prisma.piece.create({ data: data.piece });
  // Jika operasi kedua gagal, database berada dalam status inkonsisten (piece ada tapi diamond gagal)
  await prisma.diamond.create({ data: { ...data.diamond, pieceId: piece.id } });
}

// ❌ ANTI-PATTERN 3: Melempar error mentah ke client tanpa struktur standar
export async function badErrorHandling(input: unknown) {
  'use server';
  // Jangan biarkan error stack trace database bocor ke antarmuka klien
  throw new Error('Database connection failed at postgres://...');
}
```

---

## 6. Aturan & Invariants (Rules)

1. **Direktif `'use server'` Wajib:** Setiap file Server Action wajib diawali dengan direktif `'use server'`.
2. **Validasi Skema Zod Mutlak:** Seluruh parameter input harus bertipe `unknown` pada signature fungsi dan divalidasi menggunakan `Schema.safeParse()` sebelum diproses.
3. **Pemeriksaan Sesi & Peran Server-Side:** Seluruh mutasi sensitif (terutama `/admin`) wajib mengecek `await auth()` dan memeriksa `session.user.role`. Jangan pernah mengandalkan middleware routing saja.
4. **Transaksi Atomik Multi-Tabel:** Setiap mutasi yang melibatkan lebih dari satu operasi tabel wajib menggunakan `prisma.$transaction()`.
5. **Revalidasi Jalur Cache:** Panggil `revalidatePath()` secara eksplisit pada rute halaman yang terpengaruh agar perubahan data segera tercermin di sisi pengguna publik.
6. **Bebas Tipe `any`:** Seluruh tipe masukan, kembalian, dan objek domain harus memiliki tipe TypeScript yang kuat dan ketat.

---

## 7. Pola & Dokumen Terkait

- [PRISMA_MODEL_PATTERN.md](./PRISMA_MODEL_PATTERN.md) — Pola pendefinisian entitas dan relasi basis data
- [API_RESPONSE_PATTERN.md](./API_RESPONSE_PATTERN.md) — Standardisasi respon envelope
- [CODING_RULES.md](../CODING_RULES.md) — Konvensi kode proyek
