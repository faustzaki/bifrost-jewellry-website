# DECISIONS/ — Rekaman Keputusan Arsitektur (Architectural Decision Records)

> **Tujuan:** Direktori ini mendokumentasikan seluruh Keputusan Arsitektur (*Architectural Decision Records / ADR*) penting yang diambil selama perancangan dan pengembangan sistem Bifrost Jewellery. Setiap keputusan mencatat konteks masalah, alasan pemilihan solusi, alternatif yang ditolak, serta konsekuensinya.

---

## Indeks Rekaman Keputusan (ADR Index)

| ADR | Judul Keputusan | Status | Tanggal | Area Sistem |
|---|---|---|---|---|
| [ADR 001](./001-nextjs15-app-router-postgres.md) | Arsitektur Modular Monolith Next.js 15 App Router & PostgreSQL | `Accepted` | 2026-10-11 | Arsitektur & Backend |
| [ADR 002](./002-quiet-luxury-design-direction.md) | Arah Desain Visual Editorial Quiet Luxury & Aturan Tipografi | `Accepted` | 2026-10-11 | Desain & Frontend |
| [ADR 003](./003-hybrid-commerce-model.md) | Model Perdagangan Hibrida: Checkout Online Langsung & Janji Temu Butik | `Accepted` | 2026-10-11 | Bisnis & Alur Transaksi |
| [ADR 004](./004-guest-checkout-optional-account.md) | Dukungan Penuh Guest Checkout dengan Pembuatan Akun Opsional | `Accepted` | 2026-10-11 | Checkout & Autentikasi |
| [ADR 005](./005-dual-origin-natural-lab-grown.md) | Penawaran Ganda Berlian Alami & Lab-Grown dengan Transparansi Sertifikat | `Accepted` | 2026-10-11 | Katalog & Etika Produk |
| [ADR 006](./006-midtrans-snap-payments.md) | Integrasi Pembayaran Midtrans Snap & Opsi Transfer Bank Manual | `Accepted` | 2026-10-11 | Pembayaran & Keuangan |
| [ADR 007](./007-in-app-admin-panel.md) | Panel Admin Terintegrasi In-App Menggunakan Server Actions & RBAC | `Accepted` | 2026-10-11 | Operasional & Admin |
| [ADR 008](./008-english-only-content.md) | Bahasa Konten Publik Bahasa Inggris Penuh & Dokumentasi Bahasa Indonesia | `Accepted` | 2026-10-11 | Konten & Tata Kelola |

---

## Panduan Penambahan Keputusan Baru

Untuk mencatat keputusan arsitektur baru:
1. Salin format template resmi dari [`_TEMPLATE.md`](./_TEMPLATE.md).
2. Berikan nomor urut berikutnya dengan format 3 digit (contoh: `009-nama-keputusan.md`).
3. Tambahkan baris baru pada tabel indeks di atas.
4. Jangan pernah menghapus keputusan lama yang telah `Accepted`. Jika suatu keputusan digantikan oleh arsitektur baru, ubah statusnya menjadi `Superseded` dan berikan tautan ke ADR pengganti.
