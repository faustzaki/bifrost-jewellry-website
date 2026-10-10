# 🤖 AGENTS.md — Master AI Agent Router & Protokol Kerja

> **Tujuan:** Dokumen ini adalah *entry point* navigasi utama bagi seluruh AI agent yang bekerja pada proyek Bifrost Jewellery. Baca dokumen ini **PERTAMA** sebelum menavigasi berkas lainnya. Dokumen ini menentukan hierarki dokumen yang wajib dimuat berdasarkan jenis tugas (*task*) yang sedang dikerjakan.

---

## 📌 13 Global Constraints (BATASAN MUTLAK WAJIB DIPATUHI)

Setiap AI agent **WAJIB** tunduk pada 13 batasan global berikut tanpa pengecualian:

1. **DILARANG MENGARANG DATA PALSU:** Dilarang menghasilkan data dummy palsu, nomor sertifikat grading fiktif, alamat karangan, atau informasi bisnis yang tidak bersumber dari dokumentasi resmi. Gunakan penanda `<TBD: ...>` jika data belum tersedia dan daftarkan di `CURRENT_STATE.md §Open Inputs`.
2. **WAJIB MEMBACA MEMORI TIER 0:** Wajib membaca `CURRENT_STATE.md` di awal sesi dan memperbaruinya di akhir sesi kerja.
3. **WAJIB MEMATUHI CODING RULES:** Seluruh output kode wajib mematuhi standar penulisan di `CODING_RULES.md` (Server Components by default, Zero Any, minor units money).
4. **WAJIB MEMATUHI ARSITEKTUR:** Seluruh penambahan kode wajib selaras dengan arsitektur modular monolith di `ARCHITECTURE.md`.
5. **WAJIB MEMATUHI DESIGN SYSTEM QUIET LUXURY:** Seluruh output UI wajib menggunakan token semantik resmi (`--ql-*`) di `UI_UX.md`. Dilarang menggunakan warna hex sembarangan atau gaya ritel mencolok.
6. **WAJIB MENGGUNAKAN GLOSARIUM RESMI:** Gunakan istilah baku dari `GLOSSARY.md`. Dilarang menggunakan istilah alias yang dilarang (*Do not use*).
7. **DILARANG MENAMBAH DEPENDENSI SEMBARANG:** Dilarang menginstal paket pustaka baru tanpa konfirmasi eksplisit dari pengguna.
8. **DILARANG MENGUBAH SKEMA DB TANPA KONSULTASI:** Dilarang mengubah skema PostgreSQL tanpa mereferensikan dan memperbarui `DATABASE.md`.
9. **DILARANG MENENTANG ADR TANPA DISKUSI:** Dilarang melanggar keputusan arsitektur yang telah terkunci di `DECISIONS/`.
10. **WAJIB REKOMPUTASI HARGA DI SISI SERVER:** Dilarang mempercayai harga dari client pada alur checkout. Server wajib menghitung ulang harga secara mutlak dari database.
11. **MUTASI ADMIN MENGGUNAKAN SERVER ACTIONS DENGAN ZOD:** Seluruh mutasi data internal admin wajib menggunakan Server Actions yang divalidasi dengan skema Zod di sisi server, bukan REST API publik.
12. **BAHASA DOKUMENTASI VS PUBLIK:** Seluruh berkas dokumentasi internal AI wajib ditulis dalam **Bahasa Indonesia**, sedangkan teks publik antarmuka website menggunakan **Bahasa Inggris** (ADR-008).
13. **ISOLASI KEAMANAN FINANSIAL & PCI-DSS:** Dilarang menangani data kartu kredit langsung pada server aplikasi. Seluruh interaksi pembayaran kartu kredit dialihkan melalui Midtrans Snap.

---

## 📂 Indeks Berkas & Hierarki Dokumentasi (Documentation Index)

### Tier 0: Sesi & Memori Kerja (Selalu Dibaca Pertama, Diperbarui Terakhir)
| Berkas | Fungsi & Kegunaan | Kapan Wajib Dibaca |
|---|---|---|
| [`CURRENT_STATE.md`](./CURRENT_STATE.md) | Snapshot status terkini, riwayat sesi, dan daftar input terbuka TBD | **Selalu** di awal dan akhir sesi |

### Tier 1: Fondasi Proyek (Wajib Dipahami Sebelum Menulis Kode)
| Berkas | Fungsi & Kegunaan | Kapan Wajib Dibaca |
|---|---|---|
| [`AGENTS.md`](./AGENTS.md) | Router navigasi dan batasan global sistem | **Selalu** sebagai entry point |
| [`PROJECT_CONTEXT.md`](./PROJECT_CONTEXT.md) | Identitas brand, target pengguna, batasan scope, dan FR/NFR | Saat memahami kebutuhan bisnis |
| [`CODING_RULES.md`](./CODING_RULES.md) | Standar teknis Next.js 15, TypeScript strict, dan konvensi | Saat mulai menulis/mereview kode |
| [`GLOSSARY.md`](./GLOSSARY.md) | Glosarium entitas bisnis, status enum, dan istilah terlarang | Saat menamai variabel/fungsi/entitas |

### Tier 2: Spesifikasi Teknis Sistem
| Berkas | Fungsi & Kegunaan | Kapan Wajib Dibaca |
|---|---|---|
| [`ARCHITECTURE.md`](./ARCHITECTURE.md) | Desain modular monolith, alur data checkout, dan keamanan | Saat merancang komponen/fitur baru |
| [`STACK.md`](./STACK.md) | Daftar dependensi, font resmi, dan environment variables | Saat setup, konfigurasi, atau install |
| [`DATABASE.md`](./DATABASE.md) | ERD 17 tabel PostgreSQL, indeks performa, dan migrasi Prisma | Saat bekerja dengan query atau model DB |
| [`API_CONTRACT.md`](./API_CONTRACT.md) | Spesifikasi endpoint publik, webhook Midtrans, dan envelope | Saat membuat atau mengonsumsi API |
| [`UI_UX.md`](./UI_UX.md) | Token warna quiet luxury, skala tipografi, motion, dan copy rules | Saat membuat atau mengubah tampilan UI |
| [`MANUAL.md`](./MANUAL.md) | Panduan tata kelola, aturan DO/DON'T, dan FAQ dokumentasi | Saat memelihara folder `.AGENTS/` |

### Tier 3: Keputusan, Tugas, & Panduan Prompt
| Direktori | Berkas Utama | Kapan Wajib Dibaca |
|---|---|---|
| [`DECISIONS/`](./DECISIONS/README.md) | ADR 001 s/d 008 (Arsitektur, Desain, Model Hibrida, Pembayaran) | Saat ingin memahami alasan di balik keputusan |
| [`TASKS/`](./TASKS/TODO.md) | `TODO.md`, `ROADMAP.md`, `BUGS.md` | Saat merencanakan tugas atau memperbaiki bug |
| [`PROMPTS/`](./PROMPTS/UI_PROMPT.md) | `UI_PROMPT.md`, `BACKEND_PROMPT.md`, `REFACTOR_PROMPT.md` | Template panduan eksekusi sub-agent |
| *Direktori Pendukung* | `FEATURES/`, `FLOWS/`, `PATTERNS/` | Spesifikasi detail fitur, alur pengguna, dan pola kode |

---

## 🔀 Pohon Routing Pengambilan Keputusan (Routing Logic)

Gunakan alur keputusan berikut untuk memilih dokumen acuan yang tepat:

```
MULAI (Selalu baca: CURRENT_STATE.md → AGENTS.md → PROJECT_CONTEXT.md)
│
├─ Task = Mengembangkan Tampilan Frontend / Antarmuka UI?
│  → Baca: CODING_RULES.md → UI_UX.md → STACK.md → PROMPTS/UI_PROMPT.md
│
├─ Task = Mengembangkan Logika Backend / Server Actions / API?
│  → Baca: CODING_RULES.md → API_CONTRACT.md → DATABASE.md → ARCHITECTURE.md → PROMPTS/BACKEND_PROMPT.md
│
├─ Task = Memanipulasi Database / Skema Prisma / Migrasi?
│  → Baca: DATABASE.md → GLOSSARY.md → ARCHITECTURE.md
│
├─ Task = Alur Pembayaran / Checkout / Midtrans?
│  → Baca: API_CONTRACT.md → ARCHITECTURE.md → DECISIONS/006-midtrans-snap-payments.md
│
├─ Task = Sistem Janji Temu Butik / Booking?
│  → Baca: API_CONTRACT.md → DATABASE.md → DECISIONS/003-hybrid-commerce-model.md
│
├─ Task = Refactoring Kode?
│  → Baca: CODING_RULES.md → ARCHITECTURE.md → PROMPTS/REFACTOR_PROMPT.md
│
├─ Task = Melacak Progress / Memperbaiki Bug?
│  → Baca: TASKS/BUGS.md → TASKS/TODO.md → CURRENT_STATE.md
│
└─ Task = Memeriksa Keputusan Desain / Arsitektur?
   → Baca: DECISIONS/README.md → file ADR spesifik
```

---

## 📝 Protokol Pembaruan Dokumentasi (Update Protocol)

Setiap kali melakukan perubahan dalam repositori:
1. **Perubahan Skema DB:** Wajib memperbarui `DATABASE.md` dan diagram ERD terkait.
2. **Perubahan Endpoint:** Wajib memperbarui `API_CONTRACT.md` (request/response format).
3. **Keputusan Arsitektur Baru:** Wajib membuat file ADR baru di `DECISIONS/` menggunakan format `_TEMPLATE.md` dan menambahkan entri di `DECISIONS/README.md`.
4. **Penyelesaian Fitur:** Wajib memperbarui status di `TASKS/TODO.md` dan mencatat ringkasan di `CURRENT_STATE.md`.
5. **Penemuan Data TBD:** Jika nilai bisnis terisi, hapus marker dari dokumen terkait dan update `CURRENT_STATE.md §Open Inputs`.

---

## 🚀 Urutan Baca Pengenalan Pertama Kali (First-Time Read Order)

Bagi pengembang atau AI agent baru yang pertama kali menangani repositori ini, baca dokumen dengan urutan terstruktur berikut:
1. `CURRENT_STATE.md` (Ketahui status real-time terkini)
2. `PROJECT_CONTEXT.md` (Pahami apa dan untuk siapa proyek ini)
3. `GLOSSARY.md` (Pahami bahasa dan status domain)
4. `CODING_RULES.md` (Ketahui batas aturan teknis penulisan kode)
5. `STACK.md` (Pahami pustaka dan alat yang digunakan)
6. `ARCHITECTURE.md` (Pahami struktur modular monolith dan alur data)
7. `UI_UX.md` (Pahami filosofi visual dan token quiet luxury)
8. `DATABASE.md` (Pahami struktur data 17 tabel PostgreSQL)
9. `API_CONTRACT.md` (Pahami kontrak komunikasi data)
10. `DECISIONS/` (Pahami latar belakang keputusan arsitektur)

---

## 🔗 Dokumen Terkait

- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md)
- [CURRENT_STATE.md](./CURRENT_STATE.md)
- [CODING_RULES.md](./CODING_RULES.md)
- [MANUAL.md](./MANUAL.md)
