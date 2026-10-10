# 🏛️ ADR 001: Arsitektur Modular Monolith Next.js 15 App Router & PostgreSQL

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Lead Architect & Engineering Team  
> **Keputusan Terkait:** Tidak ada (ADR Fondasi)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Bifrost Jewellery membutuhkan platform e-commerce butik perhiasan mewah dengan standar visual editorial tinggi (*quiet luxury*), performa rendering halaman mendekati instan, optimasi mesin pencari (SEO) kelas dunia untuk kata kunci bernilai tinggi, serta keandalan transaksi finansial tanpa toleransi kesalahan. Sistem harus mampu menangani kurasi koleksi, pencarian multi-kriteria 4Cs berlian, pemesanan janji temu butik fisik, dan alur pembayaran online dalam satu ekosistem yang terintegrasi.

Tantangan arsitektural utama mencakup:
1. Menghindari latensi jaringan antar-layanan (*network hops*) yang dapat memperlambat rendering halaman editorial.
2. Memastikan tipe data yang aman dari antarmuka pengguna hingga ke baris database (*end-to-end type safety*).
3. Mengelola data relasional yang kompleks (relasi antara perhiasan, berlian, varian cincin, pesanan, dan verifikasi transfer manual).

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk mengimplementasikan **Modular Monolith** berbasis **Next.js 15 (App Router)** dengan **PostgreSQL 16** dan **Prisma ORM**.

Pilar implementasi keputusan ini:
- **Full-Stack Next.js 15:** Menggabungkan antarmuka React Server Components (RSC) dengan logika bisnis backend (Server Actions dan Route Handlers) dalam satu repositori (*single codebase*).
- **PostgreSQL 16 Relasional:** Menyimpan seluruh data transaksional perhiasan, inventori, sertifikat grading lab, dan status pesanan dengan integritas referensial ketat (*foreign key constraints*).
- **Prisma ORM:** Digunakan sebagai data access layer bertipe aman yang mengotomatisasi migrasi skema dan menyediakan tipe data TypeScript otomatis.
- **Edge Caching & ISR:** Memanfaatkan Incremental Static Regeneration untuk halaman katalog dan koleksi guna menjamin waktu muat sub-detik di jaringan global.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Monolith Tradisional Berbasis PHP / Laravel
- **Kelebihan:** Ekosistem mature, Blade/Livewire cepat untuk CRUD admin, antrean bawaan (*built-in queues*).
- **Alasan Ditolak:** Interaksi antarmuka editorial mewah (smooth scroll Lenis, transisi galeri Framer Motion, micro-interactions) jauh lebih optimal dan alami dibangun di atas ekosistem React. Pemisahan rendering SSR React pada arsitektur hybrid lebih rumit jika menggunakan Laravel tanpa Inertia, dan tim memprioritaskan TypeScript end-to-end.

### Alternatif B: Decoupled Headless (Next.js Frontend + Laravel / Go Backend Terpisah)
- **Kelebihan:** Pemisahan repositori frontend dan backend secara independen.
- **Alasan Ditolak:** Menambah *operational complexity* (dua repositori terpisah, dua pipeline deployment, kontrak API ganda yang harus disinkronkan). Untuk skala butik perhiasan tunggal, arsitektur decoupled adalah *over-engineering* yang tidak perlu (*premature optimization*).

### Alternatif C: Astro + Headless CMS (Sanity / Strapi)
- **Kelebihan:** Sangat cepat untuk situs konten statis.
- **Alasan Ditolak:** Kurang fleksibel untuk logika transaksi e-commerce hibrida, alur checkout real-time, manajemen state keranjang belanja dinamis, dan panel admin internal terintegrasi.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **End-to-End Type Safety:** Skema Prisma langsung menghasilkan tipe data TypeScript yang dapat digunakan di Server Actions maupun komponen UI.
- **Performa LCP Maksimal:** Halaman detail perhiasan (PDP) dapat di-render di server secara statis (SSG/ISR) dengan payload HTML murni, meminimalkan JavaScript bundle di browser.
- **Zero API Boilerplate:** Mutasi data internal admin dan keranjang belanja menggunakan Server Actions tanpa perlu menyusun puluhan rute REST API tambahan.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Manajemen Koneksi Database Serverless:** Karena Next.js berjalan di lingkungan serverless/edge Vercel, koneksi PostgreSQL memerlukan connection pooling (PgBouncer atau Prisma Accelerate) untuk mencegah kehabisan koneksi database.
- **Beban Kognitif Server vs Client Components:** Pengembang wajib memahami batasan pemisahan Server Components dan Client Components secara disiplin.

---

## 5. Referensi & Dokumen Terkait

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Diagram arsitektur modular monolith
- [STACK.md](../STACK.md) — Konfigurasi tumpukan teknologi Next.js 15 & Prisma
- [DATABASE.md](../DATABASE.md) — Skema database PostgreSQL 16
