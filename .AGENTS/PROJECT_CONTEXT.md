# PROJECT_CONTEXT.md — Identitas Proyek & Batasan Scope

> **Tujuan:** Dokumen ini mendefinisikan identitas proyek secara menyeluruh. AI agent wajib membaca file ini untuk memahami apa yang sedang dibangun, untuk siapa sistem ini dirancang, serta batasan scope fungsional dan teknisnya.

---

## 1. Identitas Proyek

- **Nama Proyek:** Bifrost Jewellery
- **Kode Proyek:** `bifrost-jewellery-website`
- **Versi:** `v0.1.0 (Planning)`
- **Status:** Planning — Penyusunan Fondasi & Dokumentasi
- **Repository URL:** `https://github.com/faustzaki/bifrost-jewellry-website.git`
- **Domain Target:** `<TBD: domain>`
- **Terakhir Diperbarui:** 2026-10-11
- **Arah Desain:** Quiet Luxury Editorial (Visual editorial mewah, hening, proporsional, dan elegan)

---

## 2. Deskripsi Proyek

Bifrost Jewellery adalah butik perhiasan berlian mewah (*fine diamond jewellery*) online yang memadukan keindahan karya seni tinggi dengan teknologi modern. Platform ini mengusung filosofi **quiet luxury**: kemewahan sejati terpancar melalui kesederhanaan desain (*restraint*), ruang kosong yang bernafas (*negative space*), kualitas material tanpa kompromi, dan transparansi penuh asal-usul batu permata (*provenance*).

Bifrost Jewellery menawarkan kurasi perhiasan berlian dengan dua kategori asal:
1. **Natural Diamonds:** Berlian alami bersertifikat resmi internasional (GIA / IGI) dengan keaslian geologis teruji.
2. **Lab-Grown Diamonds:** Berlian hasil kreasi laboratorium berstandar identik secara optik, kimiawi, dan fisik, bersertifikat resmi (IGI / GIA), dengan jejak keberlanjutan modern.

Platform ini mengadopsi model perdagangan hibrida (**Hybrid Commerce**):
- **Pembelian Online Langsung:** Pelanggan dapat memilih produk jadi (*ready-to-wear*) atau perhiasan pesanan khusus (*made-to-order*), memasukkannya ke keranjang, dan menyelesaikan checkout instan menggunakan payment gateway Midtrans Snap atau transfer bank manual.
- **Konsultasi Privat & Janji Temu Butik:** Pelanggan yang menginginkan pengalaman personal eksklusif dapat memesan sesi janji temu privat di butik fisik atau mengirimkan formulir konsultasi untuk perhiasan custom (*high jewellery bespoke*).

Sistem publik dirancang dengan copy berbahasa Inggris penuh (sesuai citra internasional), sementara dokumentasi arsitektur dan sistem panduan AI disusun dalam Bahasa Indonesia untuk efisiensi tim lokal.

---

## 3. Target Pengguna

| Peran Pengguna | Deskripsi Karakteristik | Kebutuhan Utama & Ekspektasi |
|---|---|---|
| **Visitors / Shoppers** | Calon pembeli perhiasan mewah, pencari cincin pertunangan, kado eksklusif, atau kolektor perhiasan. | Penjelajahan katalog halus, filter 4Cs presisi, transparansi nomor sertifikat, foto resolusi tinggi, panduan ukuran cincin, kemudahan guest checkout. |
| **Account Holders** | Pelanggan terdaftar yang ingin menyimpan data pribadi dan riwayat transaksi. | Akses riwayat pesanan, nomor resi pengiriman, daftar keinginan (*wishlist*), manajemen alamat tersimpan, kemudahan re-order. |
| **Boutique Staff** | Tim konsultan perhiasan di butik fisik. | Kalender jadwal janji temu harian, informasi preferensi perhiasan yang diminati tamu sebelum kunjungan, status konfirmasi kehadiran. |
| **Admin / Operations** | Pengelola katalog, tim logistik, verifikator pembayaran manual, dan pemilik butik. | Manajemen produk & varian cincin, data berlian & nomor sertifikat, verifikasi transfer bank, pembaruan status pesanan, pengaturan slot booking. |

---

## 4. Kebutuhan Bisnis (Business Requirements)

### Functional Requirements (Kebutuhan Fungsional)

- **`FR-001` Penjelajahan Katalog & Koleksi:** Menampilkan daftar koleksi perhiasan kurasi editorial dan grid katalog produk dengan filter kategori (Rings, Necklaces, Bracelets, Earrings).
- **`FR-002` Pencarian & Filter Berlian:** Fitur filter mendalam berdasarkan standar 4Cs (Carat, Cut, Color, Clarity), bentuk (*shape*), asal (*origin*: natural vs lab-grown), dan rentang harga.
- **`FR-003` Halaman Detail Produk (PDP):** Menampilkan perhiasan lengkap dengan galeri gambar multi-angle, spesifikasi logam, detail 4Cs berlian, nomor sertifikat grading (GIA/IGI) beserta tautan verifikasi, dan panduan ukuran cincin (*ring size guide*).
- **`FR-004` Keranjang Belanja (Cart):** Pengelolaan item keranjang yang mendukung pengunjung tamu (*guest*) melalui cookie/local storage serta sinkronisasi otomatis saat login.
- **`FR-005` Checkout Hibrida:** Pilihan jalur checkout mandiri dengan kalkulasi ulang harga mutlak di server, ATAU tombol sekunder "Request a Consultation" untuk pemesanan personal.
- **`FR-006` Pembayaran Transfer Bank Manual:** Opsi pembayaran transfer rekening manual dengan alur unggah bukti transfer dan verifikasi manual oleh admin (`awaiting_verification`).
- **`FR-007` Pelacakan Status Pesanan:** Antarmuka publik untuk memeriksa status pesanan menggunakan kombinasi nomor pesanan (`order_number`) dan alamat email pelanggan.
- **`FR-008` Akun Pelanggan Opsional:** Fitur registrasi dan otentikasi opsional pasca-pembelian, manajemen alamat tersimpan, wishlist, dan riwayat pesanan.
- **`FR-009` Pemesanan Janji Temu Butik (Appointment Booking):** Sistem pemesanan jadwal kunjungan privat ke butik fisik dengan pemilihan slot waktu real-time dan konfirmasi instan.
- **`FR-010` Pencari Lokasi Butik (Boutique Locator):** Informasi butik fisik mencakup alamat terperinci, peta lokasi interaktif, jam operasional, dan nomor kontak concierge.
- **`FR-011` Formulir Konsultasi & Inquiry:** Formulir permohonan konsultasi perhiasan kustom atau pertanyaan khusus terkait batu mulia tertentu.
- **`FR-012` Halaman Editorial & Edukasi:** Artikel jurnal kurasi (*editorial journal*), kisah filosofi brand (*about us*), panduan 4Cs, serta panduan perawatan perhiasan (*care guide*).
- **`FR-013` Panel Admin Internal:** Dashboard terintegrasi dalam aplikasi untuk manajemen produk, berlian, inventori varian, pesanan, verifikasi pembayaran, janji temu, dan konten FAQ.
- **`FR-014` Autentikasi & RBAC:** Sistem kontrol akses berbasis peran (*Role-Based Access Control*) dengan tingkatan `super_admin`, `admin`, `customer`, dan status `guest`.
- **`FR-015` Optimasi SEO & Data Terstruktur:** Penataan metadata dinamis, OpenGraph mewah, sitemap XML, dan schema markup JSON-LD (`Product`, `Offer`, `LocalBusiness`).
- **`FR-016` Pencarian Situs (Site Search):** Pencarian teks instan untuk menemukan produk, koleksi, dan artikel edukasi dengan respon cepat.

### Non-Functional Requirements (Kebutuhan Non-Fungsional)

- **`NFR-001` Performa Web & SEO:** Skor Google Lighthouse Performance ≥ 90 di desktop dan mobile, serta skor SEO = 100.
- **`NFR-002` Aksesibilitas & Kontras:** Tingkat kepatuhan WCAG 2.1 AA dengan rasio kontras warna minimal 4.5:1 untuk teks normal (dan 3:1 untuk teks display besar), serta dukungan penuh terhadap preferensi sistem `prefers-reduced-motion`.
- **`NFR-003` Core Web Vitals:** Largest Contentful Paint (LCP) < 2.5s, Cumulative Layout Shift (CLS) < 0.1, dan Interaction to Next Paint (INP) < 200ms.
- **`NFR-004` Integritas Finansial Server-Side:** Seluruh kalkulasi harga dan total tagihan wajib dihitung ulang di sisi server (Server-Side Price Recomputation). Nilai uang disimpan dalam integer minor units (IDR satuan penuh tanpa desimal, USD dalam cents). Dilarang mempercayai harga dari payload client.
- **`NFR-005` Keamanan & Isolasi PCI-DSS:** Aplikasi tidak menyimpan maupun memproses nomor kartu kredit secara langsung. Seluruh transaksi kartu dikelola aman melalui antarmuka Midtrans Snap hosted page/modal token.

---

## 5. Batasan Scope (Scope Boundaries)

### Dalam Scope (In Scope)
- Seluruh kebutuhan fungsional `FR-001` sampai dengan `FR-016`.
- Tampilan dua mata uang: IDR sebagai mata uang utama domestik dan USD sebagai estimasi internasional.
- Integrasi Midtrans Snap untuk pembayaran otomatis (GoPay, QRIS, Virtual Account, Kartu Kredit).
- Alur verifikasi transfer bank manual untuk transaksi bernilai tinggi.
- Penyimpanan aset gambar resolusi tinggi menggunakan Vercel Blob.
- Pengiriman email transaksional pesanan dan janji temu menggunakan Resend.

### Di Luar Scope (Out of Scope)
- Aplikasi mobile native (iOS / Android terpisah) — platform berfokus pada Progressive Web Application responsif.
- Marketplace multi-vendor — sistem hanya melayani produk resmi Bifrost Jewellery.
- Virtual try-on berbasis Augmented Reality (AR) kamera 3D.
- Chatbot otomatis berbasis AI pada antarmuka publik — interaksi mengutamakan sentuhan concierge manusia.
- Sistem lelang perhiasan atau tukar-tambah (*trade-in*) barang bekas.
- Mata uang selain IDR dan USD.

---

## 6. Metrik Keberhasilan (Success Metrics)

1. **Konversi Checkout:** Rasio penyelesaian pesanan checkout mandiri mencapai ≥ 2.5% dari pengunjung keranjang belanja.
2. **Keterlibatan Janji Temu:** Tingkat kehadiran janji temu fisik di butik mencapai ≥ 85% dari total slot yang terkonfirmasi.
3. **Keandalan Transaksi:** Nol insiden manipulasi harga berkat verifikasi perhitungan harga di sisi server.
4. **Kecepatan Rendering:** Waktu muat halaman katalog dan detail produk di bawah 1.8 detik pada jaringan 4G standar.

---

## 7. Pemangku Kepentingan (Stakeholders)

- **Founder & Creative Director:** Pemilik visi brand, pengarah estetika quiet luxury editorial, dan penentu kebijakan harga.
- **Head of Operations & Concierge:** Pengawas pemenuhan pesanan, alur janji temu butik fisik, dan verifikasi transfer bank.
- **Lead Software Architect:** Penanggung jawab keandalan arsitektur Next.js 15, integritas database PostgreSQL, dan standar keamanan.
- **Editorial & Visual Stylist:** Penyedia materi narasi, spesifikasi fotografi perhiasan, dan keaslian data grading lab.

---

## 8. Rencana Timeline 6 Fase

| Fase | Nama Fase | Fokus Utama & Luaran |
|---|---|---|
| **Fase 1** | **Dokumentasi & Fondasi Arsitektur** | Penyusunan folder `.AGENTS/`, spesifikasi aturan coding, skema database, kontrak API, panduan UI/UX quiet luxury, dan ADR. |
| **Fase 2** | **Inisialisasi & Scaffolding Sistem** | Setup Next.js 15, Tailwind v4, Prisma PostgreSQL, modul utilitas moneter, sistem layout dasar, dan setup font. |
| **Fase 3** | **Katalog Produk & Mesin Pencarian Berlian** | Implementasi halaman koleksi, filter 4Cs berlian, halaman PDP dengan viewer sertifikat, dan integrasi Vercel Blob. |
| **Fase 4** | **Commerce Hibrida & Alur Pembayaran** | Implementasi keranjang belanja, checkout guest/user, integrasi Midtrans Snap, alur transfer bank manual, dan email notifikasi Resend. |
| **Fase 5** | **Janji Temu Butik & Panel Admin Internal** | Sistem kalender booking slot butik, dashboard admin produk/pesanan/janji temu, proteksi RBAC Auth.js v5. |
| **Fase 6** | **Audit Kualitas, Keamanan, & Peluncuran** | Audit Google Lighthouse (PWA/SEO/a11y), pengujian E2E Playwright, migrasi database production, dan peluncuran resmi. |

---

## 9. Dokumen Terkait

- [STACK.md](./STACK.md) — Rincian teknologi dan dependensi
- [GLOSSARY.md](./GLOSSARY.md) — Glosarium istilah bisnis dan status sistem
- [CODING_RULES.md](./CODING_RULES.md) — Standar penulisan kode Next.js dan TypeScript
- [UI_UX.md](./UI_UX.md) — Desain sistem quiet luxury dan aturan tipografi
- [ARCHITECTURE.md](./ARCHITECTURE.md) — Arsitektur sistem modular monolith
- [DATABASE.md](./DATABASE.md) — Skema database PostgreSQL dan model Prisma
- [CURRENT_STATE.md](./CURRENT_STATE.md) — Memori status terkini proyek dan daftar input TBD
