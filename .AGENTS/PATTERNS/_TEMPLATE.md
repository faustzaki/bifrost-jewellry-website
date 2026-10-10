# 📐 Pattern Template — Panduan Penulisan Pola Arsitektur

> **Tujuan:** Gunakan template ini untuk mendokumentasikan pola arsitektur atau pola implementasi kode baru dalam proyek Bifrost Jewellery. Salin seluruh isi file ini, ubah namanya menjadi `{NAMA_POLA}_PATTERN.md` (format `UPPER_SNAKE_CASE`), dan lengkapi setiap bagian dengan penjelasan serta contoh kode TypeScript nyata yang valid.

---

# {Nama Pola} Pattern

- **Applies To:** [Backend / Frontend / Both]
- **Framework / Tools:** [Next.js 15 / Prisma / Zod / Tailwind v4]
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

[Jelaskan kondisi atau skenario spesifik di mana pola arsitektur ini HARUS diterapkan:]

- Skenario 1 (contoh: Saat mengeksekusi mutasi data inventaris atau transaksi pembayaran)
- Skenario 2 (contoh: Saat membangun formulir publik yang berinteraksi dengan basis data)

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

[Jelaskan skenario di mana pola ini TIDAK COCOK atau berlebihan (*over-engineering*):]

- Skenario 1 (contoh: Operasi read-only statis yang cukup menggunakan React Server Component murni)
- Skenario 2 (contoh: Pengecekan data sisi klien yang tidak membutuhkan revalidasi server)

---

## 3. Struktur Blueprint (Structure)

[Jelaskan tata letak berkas, dependensi, dan hierarki folder untuk pola ini:]

```
src/
└── app/
    └── {feature}/
        ├── actions.ts          # Server Action mutasi
        ├── schemas.ts          # Zod input validation schema
        └── page.tsx            # Komponen konsumsi
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

[Tulis contoh kode TypeScript Next.js 15 / Prisma / Zod yang lengkap, bertipe ketat, dan siap pakai tanpa stub atau TODO kosong:]

```typescript
// Contoh implementasi yang benar dan lengkap
import { z } from 'zod';
import { prisma } from '@/lib/db';

export const ExampleSchema = z.object({
  field: z.string().min(1, 'Field wajib diisi'),
});

export type ExampleInput = z.infer<typeof ExampleSchema>;

export async function exampleAction(input: unknown) {
  // 1. Validasi input Zod
  const validated = ExampleSchema.parse(input);

  // 2. Operasi basis data
  const result = await prisma.example.create({
    data: { field: validated.field },
  });

  return { success: true, data: result };
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

[Tulis contoh implementasi yang SALAH atau dilarang agar AI agent dan pengembang tidak mengulangi kesalahan tersebut:]

```typescript
// JANGAN LAKUKAN INI:
// 1. Melewatkan validasi input (menggunakan 'any')
// 2. Mempercayai nilai harga yang dikirim dari klien
export async function badAction(data: any) {
  return await prisma.example.create({ data }); // Bahaya! Tidak divalidasi
}
```

---

## 6. Aturan / Invariants (Rules)

[Daftar aturan mutlak yang wajib dipatuhi saat menerapkan pola ini:]

1. **Aturan 1:** Seluruh input wajib divalidasi menggunakan Zod schema sebelum dieksekusi.
2. **Aturan 2:** Mutasi data multi-tabel wajib dibungkus dalam transaksi atomik Prisma (`prisma.$transaction`).
3. **Aturan 3:** Tidak boleh menggunakan tipe data `any`.

---

## 7. Pola & Dokumen Terkait

- [CODING_RULES.md](../CODING_RULES.md) — Aturan penulisan kode proyek
- [ARCHITECTURE.md](../ARCHITECTURE.md) — Arsitektur sistem menyeluruh
