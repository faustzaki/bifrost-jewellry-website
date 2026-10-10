# 📦 FEATURES — Indeks Spesifikasi Fitur

> **Proyek:** Bifrost Jewellery (`bifrost-jewellery-website`)  
> **Tujuan:** Direktori ini mendokumentasikan spesifikasi teknis dan fungsional setiap modul dalam platform Bifrost Jewellery. Setiap dokumen memuat status, lingkup (scope), tujuan utama, user story, aturan bisnis, integrasi data, kontrak endpoint/Server Action, komponen antarmuka, dan kriteria penerimaan yang teruji.

---

## 🗺️ Peta Fitur (Feature Map)

Seluruh fitur saat ini berstatus **Planned** (Fase Perencanaan):

| Fitur / Modul | File Spesifikasi | Lingkup (Scope) | Kebutuhan Fungsional | Status |
|---|---|---|---|---|
| **Katalog & Koleksi** | [`CATALOG.md`](./CATALOG.md) | Publik | `FR-001` | Planned |
| **Pencarian Berlian (4Cs)** | [`DIAMOND_SEARCH.md`](./DIAMOND_SEARCH.md) | Publik | `FR-002`, `FR-016` | Planned |
| **Detail Produk & Sertifikasi** | [`PRODUCT_DETAIL.md`](./PRODUCT_DETAIL.md) | Publik | `FR-003` | Planned |
| **Keranjang & Checkout** | [`CART_CHECKOUT.md`](./CART_CHECKOUT.md) | Publik (Guest + User) | `FR-004`, `FR-005` | Planned |
| **Pembayaran & Webhook** | [`PAYMENT.md`](./PAYMENT.md) | Publik + Sistem + Admin | `FR-005`, `FR-006` | Planned |
| **Manajemen Pesanan** | [`ORDERS.md`](./ORDERS.md) | Publik + User + Admin | `FR-007` | Planned |
| **Akun Pelanggan & Wishlist** | [`ACCOUNT.md`](./ACCOUNT.md) | Customer | `FR-008` | Planned |
| **Janji Temu Butik** | [`APPOINTMENT.md`](./APPOINTMENT.md) | Publik + Admin | `FR-009` | Planned |
| **Lokasi Butik Fisik** | [`BOUTIQUE.md`](./BOUTIQUE.md) | Publik | `FR-010` | Planned |
| **Konsultasi Privat & Kustom** | [`CONSULTATION.md`](./CONSULTATION.md) | Publik + Admin | `FR-011` | Planned |
| **Halaman Editorial & Panduan** | [`EDITORIAL.md`](./EDITORIAL.md) | Publik | `FR-012` | Planned |
| **Panel Admin Internal** | [`ADMIN.md`](./ADMIN.md) | Admin (`admin`, `super_admin`) | `FR-013` | Planned |
| **Otentikasi & RBAC** | [`AUTH.md`](./AUTH.md) | Sistem + Semua Peran | `FR-014` | Planned |
| **Legalitas, Kebijakan & FAQ** | [`LEGAL_AND_FAQ.md`](./LEGAL_AND_FAQ.md) | Publik | `FR-015`, `FR-016` | Planned |

---

## 🚫 Di Luar Lingkup (Out-of-Scope)

Fitur-fitur berikut secara eksplisit **TIDAK TERMASUK** dalam cakupan pengembangan:

1. **Aplikasi Mobile Native (iOS / Android):** Aplikasi dikembangkan sebagai Progressive Web App (PWA) / web responsif mobile-first modern tanpa binary native store.
2. **Multi-Vendor Marketplace:** Platform adalah butik eksklusif monobrand Bifrost Jewellery; tidak menerima vendor pihak ketiga.
3. **AR Virtual Try-On:** Tidak menyediakan fitur augmentasi kamera real-time; visualisasi mengandalkan fotografi produk studio resolusi tinggi dan panduan dimensi akurat.
4. **AI Chatbot Otomatis:** Interaksi pelanggan berfokus pada layanan concierge manusia personal via formulir konsultasi dan janji temu butik privat.
5. **Lelang & Tukar Tambah (Trade-in/Auction):** Platform hanya melayani penjualan langsung perhiasan baru dan pesanan khusus (*made-to-order*).
6. **Mata Uang Selain IDR dan USD:** Transaksi keuangan dan tampilan harga dibatasi hanya pada Rupiah Indonesia (IDR) dan Dolar Amerika Serikat (USD).

---

## 🔗 Dokumen Terkait

- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Konteks bisnis dan daftar lengkap Functional Requirements
- [ARCHITECTURE.md](../ARCHITECTURE.md) — Arsitektur teknis modular monolith Next.js 15
- [DATABASE.md](../DATABASE.md) — Skema tabel PostgreSQL dan relasi data
- [API_CONTRACT.md](../API_CONTRACT.md) — Kontrak endpoint REST dan Server Actions
- [FLOWS/README.md](../FLOWS/README.md) — Diagram alur kerja pengguna dan sistem
