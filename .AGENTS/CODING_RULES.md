# CODING_RULES.md — Standar Penulisan Kode & Konvensi Teknis

> **Tujuan:** Dokumen ini mendefinisikan aturan ketat penulisan kode untuk proyek Bifrost Jewellery. Setiap baris kode yang dihasilkan oleh pengembang maupun AI agent **WAJIB** mematuhi standar arsitektur Next.js 15, TypeScript strict, keamanan finansial, dan konvensi desain yang tercantum di sini.

---

## 1. Prinsip Rekayasa Perangkat Lunak (Core Engineering Principles)

1. **Server Components by Default:** Seluruh komponen di dalam direktori `src/app/` dan `src/components/` harus berupa React Server Component (RSC) secara bawaan. Direktif `"use client"` HANYA boleh disematkan pada daun terluar komponen (*leaf components*) yang membutuhkan hooks React (`useState`, `useEffect`, `useRef`), event listener browser, atau animasi interaktif.
2. **Kalkulasi Ulang Harga Server-Side (Server-Side Price Recomputation):** **DILARANG KERAS** mempercayai harga, diskon, atau total nilai belanja yang dikirimkan dari payload browser client. Server Actions dan Route Handlers checkout WAJIB mengambil ulang harga resmi perhiasan langsung dari database PostgreSQL dan menghitung ulang total belanja secara independen.
3. **Validasi Skema Berlapis (Strict Zod Validation):** Setiap Server Action dan API Route handler WAJIB memvalidasi input masuk menggunakan skema Zod sebelum memproses logika bisnis atau query database.
4. **Zero Any Policy:** Penggunaan tipe data `any` dilarang tanpa toleransi. Seluruh bentuk data domain wajib didefinisikan secara eksplisit di `src/lib/types/`.
5. **DRY, KISS, & Single Responsibility:** Manfaatkan fungsi utilitas bersama di `src/lib/`, hindari abstraksi berlebihan yang belum dibutuhkan (*no premature abstraction*), dan pastikan satu modul memiliki satu tanggung jawab jelas.

---

## 2. Struktur Direktori & Konvensi Berkas (File & Directory Conventions)

```
bifrost-jewellery-website/
├── prisma/
│   ├── schema.prisma           # Skema deklaratif database PostgreSQL
│   └── seed.ts                 # Seeder data awal katalog, butik, & kategori
├── public/                     # Aset publik statis (favicon, manifest, SVG statis)
├── src/
│   ├── app/                    # Next.js 15 App Router
│   │   ├── (store)/            # Route group storefront publik
│   │   │   ├── collections/    # Halaman penjelajahan koleksi
│   │   │   ├── pieces/         # Halaman katalog & PDP (/pieces/[slug])
│   │   │   ├── boutique/       # Halaman butik & booking janji temu
│   │   │   ├── cart/           # Halaman keranjang belanja
│   │   │   ├── checkout/       # Halaman alur checkout hibrida
│   │   │   └── page.tsx        # Beranda editorial
│   │   ├── admin/              # Panel admin internal (terproteksi RBAC)
│   │   ├── api/                # Route Handlers (Midtrans webhook, slots, search)
│   │   ├── layout.tsx          # Root layout aplikasi (Font loader, Lenis smooth scroll)
│   │   └── globals.css         # Styling global Tailwind v4 & deklarasi token @theme
│   ├── components/             # Reusable UI Components (kebab-case.tsx)
│   │   ├── ui/                 # Komponen atomik (button, input, badge, modal)
│   │   ├── layout/             # Navbar, footer, container editorial
│   │   └── modules/            # Komponen fitur (product-card, filter-panel, cart-drawer)
│   ├── lib/                    # Logika bisnis inti & helper
│   │   ├── actions/            # Server Actions mutasi bisnis terproteksi
│   │   ├── api/                # Helper envelope response API standar
│   │   ├── auth/               # Konfigurasi Auth.js v5 & guard RBAC
│   │   ├── db/                 # Klien Prisma singleton (`prisma.ts`)
│   │   ├── money.ts            # Utilitas pemformatan mata uang IDR & USD
│   │   ├── types/              # Deklarasi tipe TypeScript domain
│   │   └── validation/         # Skema validasi Zod untuk seluruh formulir & mutasi
└── tests/                      # Pengujian Vitest & Playwright
```

### Konvensi Penamaan Berkas & Struktur Komponen

- Berkas komponen menggunakan format **kebab-case** dengan ekstensi `.tsx`: contoh `piece-card.tsx`, `appointment-picker.tsx`.
- Berkas utilitas menggunakan format **kebab-case** dengan ekstensi `.ts`: contoh `money.ts`, `auth-guard.ts`.
- **Urutan Struktur Internal Berkas Komponen:**
  1. Direktif `"use client"` (HANYA jika wajib).
  2. Import dependensi eksternal (React, Next.js, Framer Motion, Phosphor Icons).
  3. Import dependensi internal (komponen lokal, skema Zod, helper utilitas).
  4. Import tipe data TypeScript (`import type { ... }`).
  5. Deklarasi interface / types props komponen.
  6. Fungsi komponen utama (`export function ComponentName(...)`).
  7. Komponen pembantu internal atau sub-komponen private.

---

## 3. Aturan Khusus Next.js 15 & React 19

- **Halaman (`page.tsx`) Selalu Server Component:** Dilarang menyematkan direktif `"use client"` langsung di berkas `page.tsx`. Halaman bertindak sebagai orkestrator yang mengambil data di server, lalu mengopernya ke komponen interaktif client yang terisolasi.
- **Navigasi Internal:** Selalu gunakan komponen `next/link` untuk routing internal. Dilarang menggunakan tag anchor mentah `<a>` tanpa kebutuhan khusus.
- **Optimasi Gambar (`next/image`):**
  - Seluruh gambar perhiasan wajib menggunakan `next/image`.
  - Wajib menyertakan atribut `width` dan `height` eksplisit, ATAU menggunakan properti `fill` yang dikombinasikan dengan atribut `sizes` responsif untuk mencegah terjadinya *Cumulative Layout Shift* (CLS).
  - Wajib menyertakan teks `alt` yang deskriptif untuk aksesibilitas dan SEO.
- **Metadata & SEO Dinamis:**
  - Setiap halaman publik wajib mengekspor objek `metadata` atau fungsi `generateMetadata()`.
  - Halaman produk (`/pieces/[slug]`) wajib menginjeksi schema markup JSON-LD berstandar `Product` dan `Offer`.

---

## 4. Standar Validasi Server Actions

Mutasi data admin dan transaksi komersial dilakukan melalui **Server Actions**, bukan Route Handler REST tradisional (kecuali webhook pihak ketiga).

```typescript
// Contoh Pola Resmi Server Action: src/lib/actions/cart.ts
"use server";

import { z } from "zod";
import { prisma } from "@/lib/db/prisma";
import { revalidatePath } from "next/cache";

const AddToCartSchema = z.object({
  pieceId: z.string().uuid("ID perhiasan tidak valid"),
  variantId: z.string().uuid().optional(),
  quantity: z.number().int().positive().max(5, "Maksimal pemesanan 5 buah"),
});

export async function addToCartAction(input: z.infer<typeof AddToCartSchema>) {
  // 1. Validasi Zod di sisi server
  const parsed = AddToCartSchema.safeParse(input);
  if (!parsed.success) {
    return { success: false, error: parsed.error.flatten().fieldErrors };
  }

  // 2. Eksekusi logika bisnis dengan harga dari database, bukan dari client
  try {
    // Operasi database...
    revalidatePath("/cart");
    return { success: true };
  } catch (error) {
    return { success: false, error: "Gagal menambahkan item ke keranjang" };
  }
}
```

---

## 5. Standar Styling Tailwind CSS v4 & Semantic Tokens

- **WAJIB Menggunakan Semantic Design Tokens:** Dilarang menggunakan nilai warna hex sembarangan (*arbitrary values* seperti `bg-[#f0e8d9]`) di dalam komponen UI. Seluruh styling warna wajib merujuk pada token resmi quiet luxury:

| Token CSS | Tailwind Utility Class | Peruntukan Khusus | Aturan Kontras & Penggunaan |
|---|---|---|---|
| `--ql-canvas` | `bg-ql-canvas` | Latar belakang kanvas halaman utama (`#FAF8F5`) | Latar hangat netral |
| `--ql-surface` | `bg-ql-surface` | Latar belakang kartu, modal, popover (`#FFFFFF`) | Area elevasi bersih |
| `--ql-ink` | `text-ql-ink` | Teks utama, judul, label primer (`#1A1A1A`) | Kontras 16.42:1 pada canvas (AAA) |
| `--ql-ink-muted` | `text-ql-ink-muted` | Teks pendukung, deskripsi, metadata (`#6B6259`)| Kontras 5.63:1 pada canvas (AA) |
| `--ql-line` | `border-ql-line` | Garis pembatas halus (*hairline border*) (`#E7E2DA`)| Ketebalan maksimal 1px border |
| `--ql-gold` | `text-ql-gold`, `bg-ql-gold` | Aksen ornamen & teks display besar (`#9C7C3F`)| **HANYA untuk teks ukuran ≥ 24px!** |
| `--ql-gold-deep` | `text-ql-gold-deep` | Teks aksen emas ukuran reguler/kecil (`#7C6236`)| Kontras 5.42:1 pada canvas (AA) |
| `--ql-ink-surface`| `bg-ql-ink-surface` | Blok editorial gelap pekat & footer (`#1A1A1A`)| Teks putih di atasnya 16.42:1 (AAA) |
| `--ql-success` | `text-ql-success` | Indikator sukses, status terbayar (`#15803D`) | Umpan balik status positif |
| `--ql-error` | `text-ql-error` | Indikator error, stok habis (`#B3261E`) | Umpan balik validasi & kegagalan |

- **Larangan Desain Mutlak:**
  - Dilarang menggunakan warna hitam pekat `#000000`. Gunakan `--ql-ink` (`#1A1A1A`).
  - Dilarang menggunakan token `--ql-gold` pada teks paragraf biasa (wajib gunakan `--ql-gold-deep` agar lulus WCAG AA).
  - Dilarang membuat bayangan tebal berlebihan (*heavy drop shadows*). Gunakan garis batas tipis semantik `border border-ql-line`.

---

## 6. Standar Penanganan Nilai Moneter (Money Handling Rules)

1. **Format Database Integer Minor Units:**
   - Kolom mata uang disimpan dalam satuan minor integer: `price_idr_minor` (Rupiah murni tanpa sen, misal `Rp15.000.000` disimpan `15000000`) dan `price_usd_minor` (sen dollar, misal `$1,250.00` disimpan `125000`).
   - Dilarang menggunakan tipe data `float` atau `double` untuk uang karena risiko pembulatan floating-point.
2. **Sentralisasi Pemformatan di `src/lib/money.ts`:**
   - Seluruh konversi tampilan angka ke string mata uang wajib memanggil fungsi resmi:
     - `formatIDR(minorAmount: number): string` (menghasilkan contoh: `IDR 15,000,000`)
     - `formatUSD(minorAmount: number): string` (menghasilkan contoh: `$1,250`)
     - `formatDualPrice(idrMinor: number, usdMinor: number): { idr: string; usd: string }`

---

## 7. Standar TypeScript Ketat (Strict TypeScript Policy)

- **Strict Configuration:** File `tsconfig.json` wajib mengaktifkan `"strict": true`, `"noImplicitAny": true`, dan `"strictNullChecks": true`.
- **Isolasi I/O Runtime:** Seluruh operasi I/O asinkron (pemanggilan database Prisma, pemanggilan API Midtrans, pengiriman email Resend) wajib dibungkus dalam blok `try/catch` dengan penanganan error terstruktur.
- **Isolasi Lingkungan Node:** Dilarang mengimpor modul bawaan Node.js (`node:fs`, `node:path`, `node:crypto`) ke dalam komponen yang memiliki direktif `"use client"`.

---

## 8. Aksesibilitas (Accessibility / a11y)

- **Keyboard Navigability:** Seluruh tombol interaktif, menu dropdown, modal dialog, dan elemen akordeon FAQ wajib dapat diakses dan dioperasikan menggunakan navigasi keyboard (`Tab`, `Enter`, `Escape`, `Space`).
- **Focus Rings:** Dilarang menghapus ring fokus tanpa menyediakan indikator pengganti yang jelas (`focus-visible:ring-1 focus-visible:ring-ql-gold-deep`).
- **Pemeriksaan Kontras Rasio:** Seluruh teks wajib memenuhi batas minimal rasio kontras warna WCAG AA (4.5:1 untuk teks normal, 3:1 untuk teks besar ≥ 24px).
- **Reduced Motion:** Animasi Framer Motion dan Lenis smooth scroll wajib memeriksa preferensi sistem pengguna:
  ```typescript
  const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  ```

---

## 9. Alur Kerja Git & Standar Testing

### Git Commit Conventions
Gunakan format **Conventional Commits** dengan deskripsi bahasa Inggris yang ringkas dan padat:
- `feat(catalog): add 4Cs diamond filtering by carat and clarity`
- `fix(checkout): recompute order subtotal on server before Midtrans token issuance`
- `style(ui): align typography scale with quiet luxury tokens`
- `refactor(db): optimize piece variant query with compound index`
- `test(money): add unit tests for IDR and USD minor unit formatting`

### Standar Testing
- **Unit Testing (Vitest):** Cakupan pengujian minimal 80% pada direktori logika murni `src/lib/` (terutama `money.ts`, skema validasi Zod, dan kalkulator diskon/pajak).
- **E2E Testing (Playwright):** Pengujian skenario utama end-to-end:
  1. *Happy Path Checkout:* Pelanggan menjelajahi katalog -> melihat detail perhiasan -> menambahkan ke keranjang -> menyelesaikan checkout guest -> menerima token Snap.
  2. *Appointment Booking:* Pengunjung butik memilih tanggal & slot waktu -> mengisi data kontak -> menerima konfirmasi reservasi.

---

## 10. Dokumen Terkait

- [UI_UX.md](./UI_UX.md) — Rincian lengkap token desain quiet luxury
- [STACK.md](./STACK.md) — Daftar paket dependensi dan konfigurasi framework
- [DATABASE.md](./DATABASE.md) — Struktur data dan konvensi kolom database
- [API_CONTRACT.md](./API_CONTRACT.md) — Kontrak response dan endpoint sistem
