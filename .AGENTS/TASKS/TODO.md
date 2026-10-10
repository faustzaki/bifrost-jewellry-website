# TODO.md — Daftar Tugas Operasional & Status Implementasi

> **Tujuan:** Dokumen ini melacak seluruh tugas implementasi teknis untuk proyek Bifrost Jewellery. Daftar tugas dikelompokkan berdasarkan tingkat prioritas (High, Medium, Low) serta bagian tugas yang telah terselesaikan (Completed). Setiap tugas merujuk pada kebutuhan fungsional (FR) dan spesifikasi arsitektur terkait.

---

## 🔴 Prioritas Tinggi (High Priority) — Fase 2: Scaffolding & Setup Sistem

- [ ] **Setup Repositori & Kerangka Kerja Next.js 15:** Inisialisasi proyek Next.js 15 App Router dengan TypeScript strict mode dan ESLint.
- [ ] **Konfigurasi Styling Tailwind CSS v4 & Semantic Tokens:** Setup blok `@theme` di `src/app/globals.css` dengan token semantik resmi (`--ql-canvas`, `--ql-surface`, `--ql-ink`, `--ql-ink-muted`, `--ql-line`, `--ql-gold`, `--ql-gold-deep`, `--ql-ink-surface`, `--ql-success`, `--ql-error`).
- [ ] **Konfigurasi Tipografi & Font Loader:** Setup font `Cormorant Garamond`, `Inter`, dan `JetBrains Mono` di `src/app/layout.tsx`.
- [ ] **Inisialisasi Skema Database Prisma (PostgreSQL 16):** Tulis berkas `prisma/schema.prisma` yang mencakup 17 entitas data sesuai `DATABASE.md` dan jalankan migrasi awal.
- [ ] **Implementasi Utilitas Moneter (`src/lib/money.ts`):** Buat fungsi pemformatan bilangan bulat minor unit IDR dan USD serta tulis unit test Vitest dengan coverage 100%.
- [ ] **Integrasi Smooth Scroll Lenis:** Konfigurasi wrapper Lenis di root layout dengan penanganan preferensi `prefers-reduced-motion`.

---

## 🟡 Prioritas Sedang (Medium Priority) — Fase 3 & 4: Katalog & Commerce Hibrida

- [ ] **Komponen UI Atomik & Kartu Perhiasan:** Implementasi `Button`, `Badge`, `Modal`, dan `ProductCard` dengan rasio editorial 3:4 atau 4:5 di `src/components/ui/`.
- [ ] **Halaman Katalog & Filter Berlian 4Cs (`FR-001`, `FR-002`):** Implementasi halaman `/pieces` dengan filter multi-faceted: origin (natural vs lab-grown), bentuk berlian, rentang karat, dan harga.
- [ ] **Halaman Detail Perhiasan / PDP (`FR-003`):** Implementasi `/pieces/[slug]` dengan galeri foto multi-angle, viewer nomor sertifikat GIA/IGI, tabel spesifikasi 4Cs, panduan ukuran cincin, tombol utama "Add to Bag", dan tombol sekunder "Book a Private Viewing".
- [ ] **State Keranjang Belanja (`FR-004`):** Implementasi keranjang belanja berbasis cookie/session token yang mendukung pengunjung tamu (*guest*) dan sinkronisasi akun pengguna.
- [ ] **Alur Checkout & Revalidasi Harga Server-Side (`FR-005`):** Implementasi endpoint `POST /api/checkout` yang menghitung ulang total harga secara mutlak dari database dan menerbitkan token Snap Midtrans.
- [ ] **Integrasi Gateway Midtrans Snap & Webhook Handler (`FR-005`, `NFR-005`):** Setup komponen modal Snap di frontend dan route handler `POST /api/payments/midtrans/webhook` dengan verifikasi signature SHA-512 dan idempotensi status.
- [ ] **Alur Pembayaran Transfer Bank Manual (`FR-006`):** Implementasi formulir unggah bukti transfer dan endpoint `POST /api/payments/manual-transfer` menuju status `awaiting_verification`.
- [ ] **Pengiriman Email Transaksional Resend:** Buat template email konfirmasi pesanan resmi dan bukti reservasi janji temu.

---

## 🟢 Prioritas Rendah (Low Priority) — Fase 5: Janji Temu & Panel Admin

- [ ] **Pencari Lokasi Butik & Kalender Janji Temu (`FR-009`, `FR-010`):** Implementasi halaman `/boutique` dengan informasi lokasi, peta Google Maps, dan kalender pemilihan slot janji temu.
- [ ] **Formulir Permohonan Konsultasi Kustom (`FR-011`):** Implementasi modal/halaman `/inquire` untuk perhiasan custom (*bespoke*).
- [ ] **Pelacakan Status Pesanan Publik (`FR-007`):** Implementasi halaman `/orders/track` berdasarkan order number dan email.
- [ ] **Halaman Editorial & Kisah Craftsmanship (`FR-012`):** Implementasi halaman `/journal`, `/about`, dan `/care-guide`.
- [ ] **Panel Admin Internal Terproteksi RBAC (`FR-013`, `FR-014`):** Implementasi dashboard admin `/admin` untuk manajemen katalog produk, penambahan data berlian, pembaruan status pesanan, verifikasi bukti transfer bank manual, dan jadwal janji temu.
- [ ] **Optimasi SEO & Data Terstruktur JSON-LD (`FR-015`):** Injeksi schema markup `Product`, `Offer`, dan `LocalBusiness` pada halaman terkait.

---

## ✅ Selesai (Completed) — Fase 1: Dokumentasi & Fondasi Arsitektur

- [x] **Penyusunan Seluruh Dokumen Fondasi `.AGENTS/` (2026-10-11):**
  - [x] `PROJECT_CONTEXT.md` (Identitas, 16 FR, 5 NFR, Scope, Stakeholders, Timeline)
  - [x] `STACK.md` (Next.js 15, PostgreSQL 16, Prisma, Tailwind v4, Zod, Midtrans, Resend)
  - [x] `GLOSSARY.md` (Piece, Diamond, 4Cs, Certificate, Setting, status enum, istilah terlarang)
  - [x] `CODING_RULES.md` (Server Components, revalidasi harga server, TypeScript zero any, minor units)
  - [x] `UI_UX.md` (Quiet luxury design system, token warna WCAG AA, tipografi, copywriting rules)
  - [x] `ARCHITECTURE.md` (Modular monolith, diagram arsitektur mermaid, alur sequence checkout, keamanan)
  - [x] `DATABASE.md` (Skema komprehensif 17 tabel PostgreSQL, ERD, indeks performa, aturan migrasi)
  - [x] `API_CONTRACT.md` (Route Handlers, standard response envelopes, Midtrans webhook, rate limiting)
  - [x] `CURRENT_STATE.md` (Snapshot memori AI, §Open Inputs berisi seluruh penanda TBD)
  - [x] `AGENTS.md` (13 Global Constraints, tabel routing, protokol pembaruan)
  - [x] `MANUAL.md` (Panduan pemeliharaan folder `.AGENTS/`, hierarki tier, FAQ)
- [x] **Penyusunan Paket Keputusan Arsitektur `DECISIONS/` (2026-10-11):**
  - [x] `README.md` & `_TEMPLATE.md`
  - [x] ADR 001 s/d ADR 008 berstatus `Accepted`.
- [x] **Penyusunan Berkas Manajemen Tugas & Prompt (2026-10-11):**
  - [x] `TASKS/TODO.md`, `TASKS/ROADMAP.md`, `TASKS/BUGS.md`
  - [x] `PROMPTS/UI_PROMPT.md`, `PROMPTS/BACKEND_PROMPT.md`, `PROMPTS/REFACTOR_PROMPT.md`
