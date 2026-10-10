# ROADMAP.md — Rencana Kerja Strategis & Peta Jalan 6 Fase

> **Tujuan:** Dokumen ini menetapkan peta jalan (*roadmap*) jangka panjang pengembangan sistem Bifrost Jewellery dari fase perencanaan arsitektur hingga peluncuran publik. Dokumen ini mendefinisikan tonggak pencapaian (*milestones*), kriteria keberhasilan per fase, dan riwayat versi sistem.

---

## 1. Ikhtisar Peta Jalan (Roadmap Overview)

Proyek Bifrost Jewellery dikembangkan melalui **6 Fase Terstruktur** dengan pendekatan bertahap (*incremental delivery*):

```
Fase 1: Dokumentasi & Fondasi Arsitektur ──────► (SELESAI - v0.1.0)
Fase 2: Inisialisasi & Scaffolding Sistem ─────► (FASE BERIKUTNYA)
Fase 3: Katalog Produk & Filter Berlian ──────► (PLANNED)
Fase 4: Commerce Hibrida & Pembayaran ────────► (PLANNED)
Fase 5: Janji Temu Butik & Panel Admin ───────► (PLANNED)
Fase 6: Audit Kualitas & Peluncuran Publik ────► (PLANNED)
```

---

## 2. Rincian Fase Pengembangan (Phase Breakdown)

### Fase 1: Dokumentasi & Fondasi Arsitektur (Status: COMPLETED ✅)
- **Fokus Utama:** Penyusunan fondasi sistem lengkap, standarisasi aturan penulisan kode, perancangan basis data, sistem desain quiet luxury, dan penguncian keputusan arsitektur (ADR).
- **Luaran (Deliverables):**
  - Seluruh paket dokumentasi `.AGENTS/` (11 berkas root).
  - Paket ADR 001 s/d ADR 008 lengkap di `DECISIONS/`.
  - Manajemen tugas dan template prompt di `TASKS/` dan `PROMPTS/`.
- **Kriteria Sukses:**
  - 100% berkas dokumentasi konsisten tanpa konflik spesifikasi.
  - Seluruh penanda data bisnis yang belum ditentukan tercatat di `CURRENT_STATE.md §Open Inputs`.
  - Tim pengembang dan AI agent dapat bekerja tanpa memerlukan re-penjelasan konsep dasar.

---

### Fase 2: Inisialisasi & Scaffolding Sistem (Status: UPCOMING 🚀)
- **Fokus Utama:** Pengaturan repositori, setup framework Next.js 15, konfigurasi Tailwind v4 semantik, setup Prisma ORM, dan utilitas moneter.
- **Luaran (Deliverables):**
  - Proyek Next.js 15 App Router siap kompilasi dengan TypeScript strict.
  - File `src/app/globals.css` dengan token tema semantik `--ql-*`.
  - Integrasi font `Cormorant Garamond`, `Inter`, dan `JetBrains Mono`.
  - Skema deklaratif `prisma/schema.prisma` dan migrasi database awal.
  - Utilitas `src/lib/money.ts` dengan unit test Vitest lengkap.
  - Integrasi smooth scroll inertia via Lenis.
- **Kriteria Sukses:**
  - `npm run build` dan `npm run lint` berjalan 100% bersih tanpa error/warning.
  - Migrasi skema database berhasil diterapkan pada PostgreSQL.

---

### Fase 3: Katalog Produk & Mesin Pencarian Berlian (Status: PLANNED 📅)
- **Fokus Utama:** Antarmuka storefront publik, penjelajahan koleksi editorial, sistem filter 4Cs berlian, dan halaman detail produk (PDP).
- **Luaran (Deliverables):**
  - Halaman beranda editorial `/` dengan hero section quiet luxury dan kurasi koleksi pilihan.
  - Halaman indeks koleksi `/collections` dan halaman koleksi spesifik `/collections/[slug]`.
  - Halaman katalog filterable `/pieces` dengan filter 4Cs (origin, shape, carat, cut, color, clarity, price).
  - Halaman detail produk `/pieces/[slug]` dengan galeri gambar multi-angle, spesifikasi logam, detail nomor sertifikat GIA/IGI, panduan ukuran cincin, dan tombol aksi ganda.
- **Kriteria Sukses:**
  - Skor Lighthouse Performance pada halaman katalog ≥ 90.
  - Filter 4Cs berlian merespons instan tanpa reload penuh halaman.
  - Gambar perhiasan teroptimasi sempurna anti-CLS via `next/image`.

---

### Fase 4: Commerce Hibrida & Alur Pembayaran (Status: PLANNED 📅)
- **Fokus Utama:** Alur keranjang belanja, checkout mandiri guest/user, integrasi Midtrans Snap, transfer bank manual, dan email transaksional.
- **Luaran (Deliverables):**
  - Drawer dan halaman keranjang belanja `/cart` dengan dukungan sinkronisasi cookie tamu.
  - Alur checkout multi-step `/checkout` dengan revalidasi harga mutlak di sisi server (`POST /api/checkout`).
  - Integrasi payment gateway Midtrans Snap (modal pembayaran digital).
  - Handler webhook `POST /api/payments/midtrans/webhook` dengan verifikasi SHA-512 signature key dan idempotensi.
  - Opsi transfer bank manual dengan pengunggahan bukti bayar (`POST /api/payments/manual-transfer`).
  - Pengiriman email tanda terima pesanan otomatis via Resend.
- **Kriteria Sukses:**
  - Transaksi pembayaran simulasi Midtrans sandbox sukses dari awal hingga akhir.
  - Nol kemungkinan manipulasi total harga belanja berkat kalkulasi ulang di server.
  - Notifikasi webhook memicu pembaruan status pesanan secara akurat.

---

### Fase 5: Janji Temu Butik & Panel Admin Internal (Status: PLANNED 📅)
- **Fokus Utama:** Reservasi janji temu butik fisik, formulir konsultasi kustom, dan dashboard operasional admin internal.
- **Luaran (Deliverables):**
  - Halaman profil butik `/boutique` dengan peta Google Maps dan kalender booking janji temu interaktif.
  - Endpoint slot waktu `GET /api/appointments/slots` dan pendaftaran reservasi `POST /api/appointments`.
  - Halaman formulir konsultasi perhiasan kustom `/inquire`.
  - Halaman pelacakan pesanan publik `/orders/track`.
  - Dashboard internal `/admin` dengan proteksi Auth.js v5 RBAC:
    - Manajemen perhiasan & varian stok cincin.
    - Manajemen data berlian & nomor sertifikat GIA/IGI.
    - Pemrosesan pesanan & verifikasi transfer bank manual.
    - Kalender janji temu butik & respons inquiry.
- **Kriteria Sukses:**
  - Sistem kalender janji temu mencegah terjadinya konflik jadwal ganda (*double booking*).
  - Panel admin terproteksi 100% di sisi server dari akses non-admin.

---

### Fase 6: Audit Kualitas, Keamanan, & Peluncuran Publik (Status: PLANNED 📅)
- **Fokus Utama:** Pengujian otomatis E2E, audit keamanan finansial, audit aksesibilitas WCAG AA, dan peluncuran produksi.
- **Luaran (Deliverables):**
  - Rangkaian pengujian end-to-end Playwright untuk skenario pembelian dan booking janji temu.
  - Audit Google Lighthouse: Skor Performance ≥ 90, SEO = 100, Accessibility ≥ 90.
  - Konfigurasi domain produksi `<TBD: domain>` dan aktivasi sertifikat SSL.
  - Beralih ke kredensial produksi Midtrans (`MIDTRANS_IS_PRODUCTION=true`).
  - Seeding data resmi perhiasan dan butik fisik.
- **Kriteria Sukses:**
  - Seluruh acceptance criteria FR-001 s/d FR-016 terpenuhi 100%.
  - Seluruh NFR-001 s/d NFR-005 terverifikasi.
  - Website beroperasi stabil di lingkungan produksi.

---

## 3. Riwayat Versi Sistem (Version History)

| Versi | Tanggal | Deskripsi Perubahan |
|---|---|---|
| `v0.1.0` | 2026-10-11 | **Rilis Fondasi Perencanaan (Planning Release):** Pembentukan arsitektur sistem modular monolith, dokumentasi lengkap folder `.AGENTS/`, sistem desain quiet luxury, skema 17 tabel PostgreSQL, dan 8 rekaman keputusan arsitektur (ADR). |
| `v0.2.0` | *TBD* | Scaffolding Next.js 15, Prisma migrasi awal, dan utilitas moneter. |
| `v0.3.0` | *TBD* | Storefront katalog perhiasan dan mesin filter 4Cs berlian. |
| `v0.4.0` | *TBD* | Alur checkout hibrida, Midtrans Snap, dan transfer manual. |
| `v0.5.0` | *TBD* | Janji temu butik dan panel admin internal terproteksi RBAC. |
| `v1.0.0` | *TBD* | Peluncuran resmi produksi (*Production Go-Live*). |

---

## 4. Dokumen Terkait

- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Definisi timeline dan batasan scope
- [CURRENT_STATE.md](../CURRENT_STATE.md) — Status terkini proyek
- [TASKS/TODO.md](./TODO.md) — Daftar tugas harian operasional
- [TASKS/BUGS.md](./BUGS.md) — Log pelaporan dan penanganan bug
