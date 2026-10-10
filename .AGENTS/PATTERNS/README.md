# 📐 PATTERNS — Indeks Pola Arsitektur & Implementasi Kode

> **Proyek:** Bifrost Jewellery (`bifrost-jewellery-website`)  
> **Tujuan:** Direktori ini mendefinisikan *blueprint* implementasi teknis baku yang wajib dipatuhi oleh setiap pengembang dan AI agent saat menulis kode sumber. Setiap pola menjelaskan **bagaimana** kode diimplementasikan secara konkret dengan arsitektur bersih, bebas abstraksi berlebih (*lazy senior developer principles*), aman, dan siap produksi.

---

## 🧭 Perbedaan PATTERNS/ dan CODING_RULES.md

| CODING_RULES.md | PATTERNS/ |
|---|---|
| *Apa* aturan dan konvensi proyek | *Bagaimana* menyusun dan mengeksekusi kode nyata |
| "Gunakan Server Actions bervalidasi Zod" | "Begini blueprint kode Server Action lengkap dengan transaksi Prisma dan RBAC" |
| Standardisasi penamaan dan token desain | Cetak biru (*blueprint*) dan contoh kode TypeScript valid |

---

## 🗺️ Indeks Pola Arsitektur

Berikut adalah 6 pola arsitektur utama yang diterapkan dalam platform Bifrost Jewellery:

| Pola Arsitektur | File Spesifikasi | Berlaku Untuk | Deskripsi Singkat |
|---|---|---|---|
| **Server Action Pattern** | [`SERVER_ACTION_PATTERN.md`](./SERVER_ACTION_PATTERN.md) | Backend / Mutations | Pola eksekusi Server Action Next.js 15 dengan validasi input Zod, otorisasi RBAC, transaksi Prisma atomik, penanganan error terstandarisasi, dan revalidasi cache ISR. |
| **Prisma Model Pattern** | [`PRISMA_MODEL_PATTERN.md`](./PRISMA_MODEL_PATTERN.md) | Database / ORM | Pola pendefinisian skema model Prisma, relasi antar tabel, penegakan enum ketat, indeks performa kueri, dan aturan migrasi/seeding. |
| **Component Pattern** | [`COMPONENT_PATTERN.md`](./COMPONENT_PATTERN.md) | Frontend / UI | Pola pemisahan Server Component vs Client Island, konsumsi token desain semantik Tailwind v4, aksesibilitas keyboard (WCAG AA), dan layout editorial *quiet luxury*. |
| **API Response Pattern** | [`API_RESPONSE_PATTERN.md`](./API_RESPONSE_PATTERN.md) | Backend / Route Handlers | Pola helper standarisasi format respon JSON (`success` dan `error` envelope) dengan kode status HTTP yang konsisten. |
| **Catalog Query Pattern** | [`CATALOG_QUERY_PATTERN.md`](./CATALOG_QUERY_PATTERN.md) | Backend / Data Layer | Pola query builder Prisma dinamis dan aman untuk penyaringan katalog dan berlian 4Cs dengan pagination efisien dan indeks basis data. |
| **Payment Webhook Pattern** | [`PAYMENT_WEBHOOK_PATTERN.md`](./PAYMENT_WEBHOOK_PATTERN.md) | Backend / Integrasi | Pola penanganan webhook pembayaran Midtrans yang aman: verifikasi hash tanda tangan SHA-512, pemeriksaan idempotensi mutlak, transisi state atomik, dan pemicu notifikasi email. |

---

## 📋 Template Pembuatan Pola Baru

Jika di masa mendatang diperlukan penambahan pola arsitektur baru, gunakan template standar:
- [`_TEMPLATE.md`](./_TEMPLATE.md) — Panduan format dan struktur penulisan pola baru.

---

## 🔗 Dokumen Terkait

- [CODING_RULES.md](../CODING_RULES.md) — Aturan dan konvensi penulisan kode
- [ARCHITECTURE.md](../ARCHITECTURE.md) — Arsitektur sistem menyeluruh
- [DATABASE.md](../DATABASE.md) — Skema tabel dan relasi data PostgreSQL
