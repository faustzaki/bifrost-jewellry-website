# CURRENT_STATE.md — Status Terkini & Memori Kerja AI

> **Tujuan:** Dokumen ini berfungsi sebagai memori persisten lintas sesi bagi seluruh AI agent yang bekerja pada proyek Bifrost Jewellery. Dokumen ini wajib dibaca pertama kali pada awal sesi untuk memahami kondisi terkini, dan wajib diperbarui pada akhir setiap sesi kerja.

---

## 1. Aturan Wajib untuk AI Agent (Rules for AI Agent)

### Saat Memulai Sesi Baru:
1. **WAJIB BACA PERTAMA:** Baca berkas ini sebelum membaca file spesifikasi lainnya.
2. **Pahami Status Terakhir:** Periksa status fase, keputusan terbaru, dan item yang masih tertunda (*pending*).
3. **Patuhi Batasan Terkunci:** Jangan membuka kembali keputusan yang telah disepakati di `DECISIONS/`.

### Saat Selesai Mengerjakan Tugas (Sebelum Sesi Berakhir):
1. **WAJIB UPDATE DOKUMEN INI:** Catat ringkasan pekerjaan yang selesai, perubahan arsitektur/file, dan langkah selanjutnya di bagian paling atas riwayat sesi.
2. **Perbarui §Open Inputs:** Jika ada input bisnis yang baru ditentukan atau dihapus, perbarui daftar `<TBD: ...>`.
3. **Pertahankan Riwayat:** Jangan menghapus ringkasan sesi terdahulu — tambahkan sesi baru di bagian paling atas.

---

## 2. Snapshot Proyek (Project Snapshot)

- **Tanggal Snapshot:** 2026-10-11
- **Status Keseluruhan:** **Fase 1 (Planning & Documentation) — Fondasi Dokumentasi Selesai 100%**
- **Cabang Git (Branch):** `main`
- **Versi Proyek:** `v0.1.0 (Planning)`
- **Kondisi Runtime:** Belum ada kode runtime/aplikasi. Repositori saat ini berisi paket dokumentasi AI agent lengkap di folder `.AGENTS/`.

---

## 3. Ringkasan Sesi Terakhir (Last Session Summary)

### Sesi: 2026-10-11 — Inisialisasi Fondasi Dokumentasi AI Agent
**Pelaksana:** Lead Agent & Sub-Agent Worker
**Pekerjaan yang Diselesaikan:**
1. **Penyusunan Paket Root `.AGENTS/` (11 Berkas Fondasi):**
   - `PROJECT_CONTEXT.md` — Identitas brand, target audiens, 16 Kebutuhan Fungsional (FR-001 s/d FR-016), NFR, ruang lingkup, dan timeline 6 fase.
   - `STACK.md` — Detail teknologi Next.js 15 App Router, React 19, TypeScript strict, Tailwind v4, Prisma, PostgreSQL 16, Midtrans Snap, Resend, Vercel Blob, dan font Cormorant Garamond / Inter / JetBrains Mono.
   - `GLOSSARY.md` — Istilah entitas baku (Piece, Diamond, 4Cs, Certificate, Setting, Collection, Boutique, Appointment), aturan enum status pesanan/pembayaran/janji temu, serta daftar istilah terlarang (*Do not use*).
   - `CODING_RULES.md` — Standar Server Components, validasi Server Actions dengan Zod, revalidasi harga server-side mutlak, token Tailwind v4 semantik, TypeScript zero-any, dan minor units IDR/USD.
   - `UI_UX.md` — Sistem desain quiet luxury editorial, token warna terverifikasi kontras WCAG AA (`--ql-canvas`, `--ql-surface`, `--ql-ink`, `--ql-ink-muted`, `--ql-line`, `--ql-gold`, `--ql-gold-deep`, `--ql-ink-surface`, `--ql-success`, `--ql-error`), skala tipografi terkunci, motion Lenis, checklist aksesibilitas, dan aturan copywriting tanpa tanda seru / diskon.
   - `ARCHITECTURE.md` — Modular monolith Next.js 15, diagram mermaid arsitektur sistem dan sequence checkout-webhook Midtrans, struktur direktori `src/`, dan arsitektur keamanan RBAC.
   - `DATABASE.md` — Skema komprehensif 17 tabel PostgreSQL 16 dengan diagram mermaid ERD, constraints, indeks performa, aturan migrasi Prisma, dan strategi seeder.
   - `API_CONTRACT.md` — Kontrak route handler, format response envelope standar, spesifikasi endpoint katalog, checkout, webhook Midtrans, janji temu, dan rate limiting.
   - `CURRENT_STATE.md` — Snapshot memori AI dan pelacak daftar seluruh marker TBD bisnis.
   - `AGENTS.md` — Router navigasi utama AI agent dengan 13 Global Constraints dan routing logic.
   - `MANUAL.md` — Panduan penggunaan, pemeliharaan, dan aturan hierarki tier folder `.AGENTS/`.
2. **Penyusunan Keputusan Arsitektur Terkunci di `DECISIONS/` (10 Berkas):**
   - Indeks ADR dan template standar.
   - 8 berkas ADR resmi: Next.js 15 & PostgreSQL (ADR-001), Arah Visual Quiet Luxury (ADR-002), Model Perdagangan Hibrida (ADR-003), Guest Checkout & Akun Opsional (ADR-004), Berlian Natural & Lab-Grown Ganda (ADR-005), Integrasi Pembayaran Midtrans Snap & Transfer Manual (ADR-006), Panel Admin Terintegrasi In-App (ADR-007), Konten Publik Berbahasa Inggris & Dokumentasi Bahasa Indonesia (ADR-008).
3. **Penyusunan Berkas Manajemen Tugas & Panduan Prompt:**
   - `TASKS/TODO.md`, `TASKS/ROADMAP.md`, dan `TASKS/BUGS.md`.
   - `PROMPTS/UI_PROMPT.md`, `PROMPTS/BACKEND_PROMPT.md`, dan `PROMPTS/REFACTOR_PROMPT.md`.

---

## 4. Langkah Selanjutnya (Next Steps)

1. **Fase 2 Scaffolding (Inisialisasi Repositori):**
   - Inisialisasi proyek Next.js 15 menggunakan TypeScript dan Tailwind v4.
   - Setup `prisma/schema.prisma` berdasarkan skema di `DATABASE.md` dan jalankan migrasi awal.
   - Buat modul utilitas moneter `src/lib/money.ts` beserta pengujian unit Vitest.
   - Setup font lokal / Google Fonts (`Cormorant Garamond`, `Inter`, `JetBrains Mono`) di `src/app/layout.tsx`.
   - Konfigurasi variabel tema semantik `--ql-*` di `src/app/globals.css`.
2. **Fase 3 Katalog & PDP:**
   - Implementasi komponen UI kartu perhiasan `piece-card.tsx` dan filter 4Cs berlian.

---

## 5. Daftar Input Terbuka (§Open Inputs)

Berikut adalah daftar lengkap seluruh nilai bisnis yang masih berstatus penanda `<TBD: ...>` di seluruh berkas dokumentasi. Dilarang mengarang data palsu; seluruh nilai di bawah ini wajib digantikan dengan data riil dari pemilik butik sebelum peluncuran produksi:

| No | Penanda TBD | Dokumen Terkait | Keterangan & Deskripsi Kebutuhan |
|---|---|---|---|
| 1 | `<TBD: domain>` | `PROJECT_CONTEXT.md`, `STACK.md`, `ARCHITECTURE.md`, `API_CONTRACT.md` | Nama domain resmi website (contoh: `https://bifrostjewellery.com`). |
| 2 | Terisi: `https://github.com/faustzaki/bifrost-jewellry-website.git` | `PROJECT_CONTEXT.md` | URL remote repositori Git resmi (GitHub). |
| 3 | `<TBD: alamat butik>` | `DATABASE.md`, `STACK.md`, `PROJECT_CONTEXT.md` | Alamat fisik lengkap butik flagship Bifrost Jewellery di Jakarta. |
| 4 | `<TBD: telepon butik>` | `DATABASE.md`, `PROJECT_CONTEXT.md` | Nomor telepon resmi telepon kabel butik untuk layanan concierge. |
| 5 | `<TBD: jam operasional butik>` | `DATABASE.md` | Jadwal jam operasional butik fisik untuk booking janji temu (contoh: Selasa–Minggu 10:00–19:00). |
| 6 | `<TBD: google maps butik>` | `DATABASE.md` | Tautan sematan resmi Google Maps untuk penunjuk lokasi butik. |
| 7 | `<TBD: whatsapp bisnis>` | `PROJECT_CONTEXT.md`, `DATABASE.md` | Nomor WhatsApp Business resmi concierge untuk konsultasi kustom. |
| 8 | `<TBD: email kontak>` | `STACK.md`, `PROJECT_CONTEXT.md` | Alamat email resmi layanan pelanggan (misal: `concierge@bifrostjewellery.com`). |
| 9 | `<TBD: entitas hukum bisnis / PT>` | `PROJECT_CONTEXT.md` | Nama resmi badan usaha hukum terdaftar di Indonesia (PT) untuk halaman Terms & Conditions. |
| 10 | `<TBD: midtrans merchant id>` | `STACK.md` | Merchant ID resmi dari akun produksi Midtrans. |
| 11 | `<TBD: midtrans server key>` | `STACK.md` | Server Key produksi Midtrans untuk otentikasi webhook & transaksi. |
| 12 | `<TBD: midtrans client key>` | `STACK.md` | Client Key produksi Midtrans untuk inisialisasi frontend Snap. |
| 13 | `<TBD: resend api key>` | `STACK.md` | Kunci API produksi dari akun Resend untuk pengiriman email. |
| 14 | `<TBD: rekening bank transfer manual>` | `PROJECT_CONTEXT.md`, `API_CONTRACT.md` | Nomor rekening resmi bank butik (BCA/Mandiri atas nama PT) untuk alur transfer manual. |
| 15 | `<TBD: katalog awal produk & berlian>` | `DATABASE.md` | Data kurasi awal produk nyata perhiasan, spesifikasi logam, dan nomor sertifikat GIA/IGI dari kurator. |

---

## 6. Dokumen Terkait

- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) — Konteks dan batasan proyek
- [AGENTS.md](./AGENTS.md) — Router navigasi utama AI Agent
- [TASKS/TODO.md](./TASKS/TODO.md) — Daftar tugas operasional
- [TASKS/ROADMAP.md](./TASKS/ROADMAP.md) — Rencana kerja jangka panjang
