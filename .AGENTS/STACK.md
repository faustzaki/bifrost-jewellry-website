# STACK.md — Tumpukan Teknologi & Dependensi

> **Tujuan:** Dokumen ini mendefinisikan seluruh ekosistem teknologi, pustaka dependensi, arsitektur data, layanan pihak ketiga, dan konfigurasi lingkungan untuk proyek Bifrost Jewellery. Setiap AI agent wajib mematuhi batasan teknologi yang tercantum di sini dan dilarang menambahkan dependensi baru tanpa persetujuan eksplisit.

---

## 1. Ikhtisar Teknologi (Stack Overview)

- **Frontend Framework:** Next.js 15 (App Router, React Server Components)
- **Backend Runtime:** Next.js Route Handlers + Server Actions terintegrasi
- **Database Engine:** PostgreSQL 16 (dengan ekstensi `uuid-ossp` atau native UUIDv4 & `citext`)
- **Bahasa Pemrograman:** TypeScript 5.x (Strict Mode diaktifkan secara penuh)
- **ORM / Data Access:** Prisma ORM 5.x / 6.x
- **State Management:** React Server Components (RSC) + Server Actions; tidak menggunakan global client-side store yang berlebihan (Zustand/Redux tidak digunakan)
- **Deployment & Hosting:** Vercel (Edge Network + Serverless Functions) & Vercel Postgres / Managed PostgreSQL

---

## 2. Dependensi Frontend & Backend (Packages Table)

| Nama Paket | Versi | Kategori | Fungsi & Penggunaan Spesifik |
|---|---|---|---|
| `next` | `^15.0.0` | Core Framework | App Router, SSR, SSG/ISR, optimasi gambar, dan Route Handlers |
| `react` | `^19.0.0` | UI Core | Pustaka rendering UI utama dengan dukungan Server Actions & RSC |
| `react-dom` | `^19.0.0` | DOM Renderer | Pengikatan React ke DOM browser |
| `typescript` | `^5.6.0` | Compiler | Type safety ketat, static analysis, dan kontrak tipe data |
| `tailwindcss` | `^4.0.0` | Styling Engine | CSS utility-first generasi terbaru dengan blok `@theme` semantik |
| `zod` | `^3.23.0` | Validasi Skema | Validasi runtime pada Server Actions, request API, dan form schema |
| `react-hook-form` | `^7.53.0` | Form State | Pengelolaan state formulir checkout, booking janji temu, dan input produk |
| `@hookform/resolvers`| `^3.9.0` | Form Resolver | Jembatan integrasi antara React Hook Form dan Zod schema |
| `framer-motion` | `^12.0.0` | Animasi & Motion | Transisi halaman halus, animasi fade editorial, dan interaksi galeri |
| `lenis` | `^1.1.0` | Smooth Scroll | Pengalaman scrolling halus mewah (*quiet luxury inertia scroll*) |
| `@phosphor-icons/react` | `^2.1.0` | Ikonografi | Set ikon minimalis seragam (weight: *regular* & *light*) |
| `next-auth` | `^5.0.0-beta` | Autentikasi | Auth.js v5 untuk otentikasi admin (Credentials) & pelanggan opsional |
| `@prisma/client` | `^6.0.0` | ORM Client | Query builder database bertipe aman untuk Server Actions & Route Handlers |
| `prisma` | `^6.0.0` | Dev Tool | Prisma CLI untuk migrasi skema database (`prisma migrate`) dan seed |
| `midtrans-client` | `^1.4.0` | Payment Gateway | Integrasi resmi Midtrans Snap API (token transaksi & verifikasi) |
| `resend` | `^4.0.0` | Email Transaksional| Pengiriman email konfirmasi order, notifikasi janji temu, dan billing |
| `sharp` | `^0.33.0` | Optimasi Citra | Pemrosesan citra performa tinggi di sisi server untuk Next Image |
| `@vercel/blob` | `^0.25.0` | Media Storage | Penyimpanan aset gambar perhiasan, berlian, dan dokumen sertifikat digital |
| `@vercel/analytics` | `^1.3.0` | Web Analytics | Pemantauan performa web dan metrik pengunjung Vercel |

---

## 3. Lapisan Data & Media (Data & Content Layer)

- **Database Relasional:** PostgreSQL 16 mengelola data transaksi transaksional murni (produk, berlian 4Cs, stok varian, pesanan, janji temu, pengguna).
- **Akses Data:** Menggunakan Prisma Client yang diisolasi di direktori `src/lib/db/`. Dilarang melakukan raw SQL queries tanpa validasi tipe data Prisma.
- **Penyimpanan Media Digital:**
  - Gambar produk multi-angle dan foto editorial disimpan di `@vercel/blob` CDN.
  - Tautan dokumen sertifikat grading lab (GIA / IGI PDF atau tautan verifikasi online) disimpan sebagai URL referensi pada tabel `diamonds`.
- **Manajemen Nilai Moneter:**
  - Nilai moneter IDR dan USD disimpan dalam format bilangan bulat terkecil (*minor units/cents*): `price_idr_minor` (satuan Rupiah bulat) dan `price_usd_minor` (sen dollar Amerika).
  - Modul pembantu `src/lib/money.ts` menjadi satu-satunya jalur pemformatan tampilan angka ke antarmuka pengguna.

---

## 4. Infrastruktur DevOps & Tooling

| Alat / Tool | Konfigurasi / Format | Tujuan Penggunaan |
|---|---|---|
| **Package Manager** | `npm` (Node.js ≥ 20.x LTS) | Pengelolaan dependensi dan eksekusi skrip build/dev |
| **Linting & Formatting** | ESLint + Prettier (Tailwind plugin) | Menegakkan kebersihan kode, konsistensi gaya penulisan, dan sorting class |
| **Testing Unit & Integrasi**| Vitest | Pengujian fungsi utilitas murni (`src/lib/money.ts`, kalkulator harga, validator Zod) |
| **Testing End-to-End** | Playwright | Pengujian skenario kritis: penelusuran katalog, alur checkout, dan pemesanan janji temu |
| **CI / CD Deployment** | Vercel GitHub Integration | Deployment otomatis pada branch `main` (Production) dan Pull Request (Preview) |

---

## 5. Layanan Pihak Ketiga (Third-Party Services)

1. **Midtrans Payment Gateway (Snap API):**
   - Layanan gerbang pembayaran utama di Indonesia.
   - Mengelola metode transaksi: Virtual Account Bank (BCA, Mandiri, BNI, BRI), QRIS, GoPay, dan Kartu Kredit dengan otentikasi 3D Secure.
   - Endpoint webhook `POST /api/payments/midtrans/webhook` menerima notifikasi status pembayaran asinkron dengan verifikasi SHA-512 signature key.
2. **Transfer Bank Manual (Jalur Operasional Internal):**
   - Disediakan khusus untuk transaksi bernilai tinggi (*high-ticket jewellery*).
   - Pengguna mentransfer dana ke rekening resmi butik dan mengunggah referensi bukti transfer.
   - Verifikasi dilakukan manual oleh tim admin/finance di panel admin.
3. **Resend (Email Gateway):**
   - Pengiriman email transaksional modern dengan template React Email.
   - Mengirimkan: tanda terima pesanan (*order confirmation*), bukti reservasi janji temu (*appointment confirmation*), dan notifikasi status pengiriman.
4. **Vercel Blob Storage:**
   - Penyimpanan berbasis edge untuk unggahan aset visual perhiasan oleh admin dengan dukungan optimasi gambar `next/image`.

---

## 6. Tipografi & Font Resmi

Sistem tipografi dirancang untuk menguatkan estetika *quiet luxury editorial*:

| Nama Font | Klasifikasi | Bobot (*Weights*) | Penggunaan Spesifik dalam Sistem |
|---|---|---|---|
| **Cormorant Garamond** | Serif Display | 400 (Regular), 500 (Medium) | Judul utama editorial (H1, H2, H3), nama koleksi, kutipan puitis, dan header produk mewah |
| **Inter** | Sans-Serif Humanis | 400 (Regular), 500 (Medium), 600 (Semi-bold) | Seluruh teks antarmuka (body text), tombol CTA, label formulir, navigasi, dan tabel spesifikasi |
| **JetBrains Mono** | Monospace Modern | 400 (Regular), 500 (Medium) | Nomor sertifikat GIA/IGI, nomor SKU produk, kode pesanan (*order number*), dan nilai moneter tabular |

---

## 7. Variabel Lingkungan (Environment Variables)

Berikut adalah daftar variabel lingkungan yang wajib dikonfigurasi pada file `.env.local` (Development) dan Vercel Project Settings (Production). **Dilarang memasukkan nilai rahasia langsung ke dalam repositori.**

```bash
# ==============================================================================
# DATABASE (PostgreSQL & Prisma)
# ==============================================================================
DATABASE_URL="postgresql://user:password@localhost:5432/bifrost_jewellery?schema=public"
DIRECT_URL="postgresql://user:password@localhost:5432/bifrost_jewellery?schema=public"

# ==============================================================================
# AUTENTIKASI (Auth.js v5 / NextAuth)
# ==============================================================================
AUTH_SECRET="<generate-random-secret-32-chars-min>"
AUTH_URL="http://localhost:3000"

# ==============================================================================
# APLIKASI WEB
# ==============================================================================
NEXT_PUBLIC_SITE_URL="http://localhost:3000"

# ==============================================================================
# PAYMENT GATEWAY (Midtrans Snap)
# ==============================================================================
MIDTRANS_SERVER_KEY="<midtrans-server-key>"
NEXT_PUBLIC_MIDTRANS_CLIENT_KEY="<midtrans-client-key>"
MIDTRANS_IS_PRODUCTION="false"

# ==============================================================================
# EMAIL TRANSAKSIONAL (Resend)
# ==============================================================================
RESEND_API_KEY="<resend-api-key>"
EMAIL_FROM="Bifrost Jewellery <concierge@bifrost-jewellery.com>"

# ==============================================================================
# MEDIA ASSETS STORAGE (Vercel Blob)
# ==============================================================================
BLOB_READ_WRITE_TOKEN="<vercel-blob-read-write-token>"
```

---

## 8. Kompatibilitas Sistem

- **Node.js Environment:** Node.js versi `20.x` LTS atau lebih baru.
- **Browser Compatibility:** Seluruh browser modern evergreen (Google Chrome, Apple Safari, Mozilla Firefox, Microsoft Edge) dengan dukungan CSS Grid dan Flexbox modern.
- **Mobile First:** Tata letak responsif dirancang dimulai dari viewport ponsel pintar (*mobile-first*) dengan transisi tanpa patahan ke tablet dan layar desktop ultrawide.

---

## 9. Dokumen Terkait

- [CODING_RULES.md](./CODING_RULES.md) — Aturan teknis penulisan kode Next.js & TypeScript
- [ARCHITECTURE.md](./ARCHITECTURE.md) — Arsitektur teknis modular monolith
- [DATABASE.md](./DATABASE.md) — Skema database dan relasi Prisma
- [UI_UX.md](./UI_UX.md) — Token visual dan styling Tailwind CSS v4
