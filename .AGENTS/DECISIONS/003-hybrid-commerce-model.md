# 🏛️ ADR 003: Model Perdagangan Hibrida: Checkout Online Langsung & Janji Temu Butik

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Founder, Head of Operations, Lead Architect  
> **Keputusan Terkait:** [ADR 001](./001-nextjs15-app-router-postgres.md), [ADR 006](./006-midtrans-snap-payments.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Perhiasan berlian memiliki spektrum nilai transaksi yang sangat luas: mulai dari karya harian (*ready-to-wear fine jewellery*) berharga belasan juta rupiah, hingga cincin pertunangan berlian 2–3 karat berharga ratusan juta rupiah (*high jewellery bespoke*).

Pola perilaku pembeli dalam kategori ini terbagi menjadi dua kelompok besar:
1. **Pembeli Mandiri (*Digital-First Shoppers*):** Pelanggan yang telah memahami spesifikasi 4Cs, mempercayai sertifikat GIA/IGI resmi, dan ingin langsung menyelesaikan pembelian secara mandiri melalui website dengan pengiriman aman berasuransi.
2. **Klien Pengalaman Eksklusif (*High-Touch Clients*):** Pelanggan yang enggan mengeluarkan dana ratusan juta rupiah tanpa melihat langsung pendaran api (*scintillation*) berlian, mencoba ukuran cincin pada jari, dan berkonsultasi privat dengan ahli perhiasan di butik fisik.

Jika platform hanya menyediakan checkout online, bisnis akan kehilangan klien bernilai tinggi. Sebaliknya, jika platform hanya menyediakan formulir konsultasi tanpa checkout mandiri, platform akan kehilangan konversi instan dari pembeli mandiri.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk mengimplementasikan **Model Perdagangan Hibrida (Hybrid Commerce Model)**.

Arsitektur sistem mendukung dua jalur konversi yang saling melengkapi:
- **Jalur A — Checkout Mandiri Online (Self-Serve Commerce):**
  - Pelanggan dapat memilih perhiasan, menentukan ukuran cincin, memasukkan ke keranjang belanja, dan menyelesaikan checkout instan via gateway Midtrans Snap atau transfer bank manual.
  - Berlaku untuk perhiasan berstatus `in_stock` (siap kirim) dan `made_to_order` (waktu tempa 2–4 minggu).
- **Jalur B — Pemesanan Janji Temu Butik & Konsultasi Kustom (Concierge & Appointment):**
  - Pada setiap halaman produk (PDP), tersedia aksi sekunder: *"Book a Private Viewing"* atau *"Request a Consultation"*.
  - Pengunjung dapat memilih tanggal dan slot waktu kosong di butik fisik resmi melalui modul kalender interaktif (`/boutique`), menentukan perhiasan spesifik yang ingin diinspeksi, dan menerima konfirmasi janji temu otomatis via email.
  - Untuk perhiasan pesanan khusus (*bespoke*), formulir konsultasi terintegrasi mencatat kebutuhan desain dan preferensi batu mulia.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Inquiry-Only / "Price Upon Request" (Tanpa Checkout Mandiri)
- **Kelebihan:** Sangat eksklusif, setiap calon pembeli disaring langsung oleh staf concierge.
- **Alasan Ditolak:** Menimbulkan friksi (*friction*) tinggi bagi pembeli modern yang ingin transaksi instan; menghilangkan potensi penjualan luar kota/luar pulau yang tidak dapat berkunjung ke butik fisik di Jakarta.

### Alternatif B: Pure Self-Serve E-Commerce (Hanya Checkout Tanpa Janji Temu)
- **Kelebihan:** Otomatisasi 100% tanpa beban operasional janji temu fisik di web.
- **Alasan Ditolak:** Mengabaikan aset strategis keberadaan butik fisik resmi; tingkat keraguan pembeli perhiasan bernilai tinggi di atas Rp50 juta akan menurunkan rasio konversi secara drastis.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Maksimalisasi Rasio Konversi:** Menjaring kedua profil pembeli (yang ingin instan maupun yang ingin pengalaman tatap muka).
- **Pemanfaatan Sinergis Butik Fisik:** Website berfungsi sebagai saluran pemasaran digital yang mengalirkan tamu berkualitas (*qualified leads*) langsung ke butik fisik.
- **Membangun Hubungan Jangka Panjang:** Interaksi tatap muka di butik memperkuat loyalitas klien perhiasan kelas atas.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Kompleksitas UI/UX PDP:** Halaman detail produk harus menampung dua tombol aksi (CTA primer untuk beli dan CTA sekunder untuk janji temu) tanpa merusak kebersihan tata letak quiet luxury.
- **Sinkronisasi Operasional:** Staf butik harus rutin memantau kalender janji temu dan menyiapkan perhiasan yang diminati sebelum tamu hadir.

---

## 5. Referensi & Dokumen Terkait

- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Definisi FR-005, FR-009, dan FR-011
- [API_CONTRACT.md](../API_CONTRACT.md) — Endpoint `/api/checkout` dan `/api/appointments`
- [DATABASE.md](../DATABASE.md) — Skema tabel `orders`, `appointments`, dan `boutiques`
