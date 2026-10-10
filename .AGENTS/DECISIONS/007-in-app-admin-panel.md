# 🏛️ ADR 007: Panel Admin Terintegrasi In-App Menggunakan Server Actions & RBAC

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Lead Architect, Engineering Team  
> **Keputusan Terkait:** [ADR 001](./001-nextjs15-app-router-postgres.md), [ADR 006](./006-midtrans-snap-payments.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Operasional harian butik Bifrost Jewellery memerlukan antarmuka administrasi internal (*back-office*) yang terintegrasi erat dengan database untuk menjalankan tugas-tugas kritis:
1. Mengelola katalog karya perhiasan, foto multi-angle di Vercel Blob, stok ukuran cincin, dan spesifikasi 4Cs berlian bersertifikat GIA/IGI.
2. Memproses pesanan masuk, memverifikasi bukti transfer bank manual, dan memperbarui nomor resi pengiriman berharga.
3. Mengelola kalender reservasi janji temu butik dan merespons permohonan konsultasi perhiasan kustom (*inquiries*).
4. Mengelola konten editorial, panduan ukuran, dan FAQ.

Jika tim menggunakan Headless CMS pihak ketiga atau aplikasi admin terpisah, sinkronisasi data relasional yang rumit (terutama siklus hidup status pesanan dan verifikasi pembayaran) akan rentan terhadap kegagalan konsistensi data.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk membangun **Panel Admin Internal Terintegrasi di Dalam Aplikasi (In-App Admin Panel)** di bawah rute `/admin`.

Karakteristik implementasi keputusan ini:
1. **Terintegrasi dalam Repositori yang Sama:** Panel admin dibangun sebagai route group Next.js 15 (`src/app/admin/`), berbagi model data Prisma dan tipe data TypeScript yang sama dengan storefront publik.
2. **Keamanan Berlapis dengan Auth.js v5 & RBAC:**
   - Seluruh halaman dan aksi di bawah `/admin` diproteksi secara ketat menggunakan session guard Auth.js v5.
   - Hak akses dibatasi hanya untuk pengguna dengan peran `super_admin` dan `admin`. Pengguna dengan peran `customer` ditolak secara otomatis (`403 Forbidden`).
3. **Mutasi Data via Server Actions dengan Validasi Zod:**
   - Seluruh penambahan, pengubahan, dan penghapusan data menggunakan **Server Actions** di `src/lib/actions/admin/`.
   - Setiap mutasi memeriksa ulang sesi login admin di sisi server dan memvalidasi payload menggunakan skema Zod sebelum menyentuh database.
4. **Desain Bersih & Fungsional:** Menggunakan sistem desain yang rapi, ergonomis untuk entri data perhiasan, serta dilengkapi pratinjau foto dan sertifikat digital.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Headless CMS Eksternal (Strapi, Sanity, atau Contentful)
- **Kelebihan:** Form builder konten siap pakai tanpa perlu koding antarmuka admin.
- **Alasan Ditolak:** Sangat baik untuk artikel blog statis, namun sangat buruk untuk siklus transaksi e-commerce (state machine pesanan, pengurangan stok varian, dan verifikasi transfer bank). Memerlukan sinkronisasi webhook yang rawan desinkronisasi data.

### Alternatif B: Aplikasi Admin Terpisah (Laravel Filament atau Django Admin)
- **Kelebihan:** Filament menyediakan generator dashboard yang sangat cepat.
- **Alasan Ditolak:** Memerlukan infrastruktur deployment kedua (PHP runtime / Python VM), konfigurasi koneksi silang ke database PostgreSQL yang sama, dan hilangnya keuntungan ekosistem tipe TypeScript terpadu.

### Alternatif C: Solusi Low-Code / SaaS Eksternal (Retool)
- **Kelebihan:** Pembuatan tabel CRUD sangat cepat melalui drag-and-drop.
- **Alasan Ditolak:** Biaya lisensi bulanan per-user yang mahal, ketergantungan pada vendor pihak ketiga (*vendor lock-in*), dan kontrol styling yang terbatas.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Konsistensi Data Mutlak (Single Source of Truth):** Server Actions di panel admin beroperasi langsung pada database PostgreSQL yang sama melalui Prisma ORM dengan transaksi atomik (`prisma.$transaction`).
- **Nol Biaya Tambahan:** Tidak ada biaya langganan software CMS eksternal.
- **Keamanan Maksimal:** Logika otorisasi peran dieksekusi langsung di lingkungan server tanpa ekspos API publik yang tidak perlu.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Upaya Pengembangan Awal (*Development Effort*):** Tim harus membangun tabel data, form input perhiasan, dan antarmuka kalender secara mandiri menggunakan komponen React.
- **Tanggung Jawab Pemeliharaan:** Setiap fitur operasional baru di masa depan harus dirawat oleh tim pengembang internal.

---

## 5. Referensi & Dokumen Terkait

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Arsitektur keamanan RBAC
- [DATABASE.md](../DATABASE.md) — Definisi peran tabel `users` (`super_admin`, `admin`)
- [CODING_RULES.md](../CODING_RULES.md) — Standar Server Actions dengan Zod
