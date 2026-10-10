# 💎 FEATURE — Detail Produk & Sertifikasi (PRODUCT_DETAIL)

> **Status:** Planned  
> **Scope:** Public Pages (Frontend + Data Layer)  
> **Primary Goal:** Menyajikan profil lengkap mahakarya perhiasan secara mendalam, meliputi galeri visual studio resolusi tinggi, tabel spesifikasi teknis 4Cs, transparansi sertifikasi resmi GIA/IGI yang dapat diverifikasi, panduan ukuran cincin (*size guide*), serta sistem tombol aksi ganda (*hybrid commerce*): transaksi langsung atau reservasi konsultasi privat.

---

## 1. Ringkasan (Overview)

Halaman Detail Produk (`/pieces/[slug]`) adalah pusat pembuktian nilai (*value proposition*) dan transparansi Bifrost Jewellery. Dalam filosofi *quiet luxury*, halaman ini tidak menggunakan taktik tekanan beli (tanpa penghitung waktu mundur / countdown timers atau lencana urgensi palsu). Sebagai gantinya, halaman ini mengedepankan pembuktian keahlian pengrajin (*craftsmanship*), kejelasan material logam mulia, dan integritas gemologi berlian.

Setiap halaman menampilkan galeri foto multi-sudut, rincian teknis berlian beserta tautan langsung ke basis data verifikasi laboratorium (GIA Report Check atau IGI Verify Report), modul interaktif panduan ukuran cincin, serta dua jalur aksi terintegrasi (*hybrid action buttons*):
1. **Primer:** "Add to Bag" — untuk pembelian daring langsung melalui keranjang dan checkout.
2. **Sekunder:** "Book a Viewing / Request Consultation" — bagi calon pembeli yang menginginkan sentuhan langsung di butik fisik atau konsultasi pesanan khusus.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Calon Pembeli:** Saya ingin melihat foto karya perhiasan dari berbagai sudut pencahayaan dan zoom detail, agar saya dapat mengamati detail potongan berlian dan kerapian rangka (*setting*).
- **Sebagai Kolektor Teliti:** Saya ingin melihat nomor laporan laboratorium GIA/IGI yang jelas dan mengklik tautan verifikasi resmi ke situs laboratorium terkait, agar keaslian dan kualitas batu berlian terkonfirmasi tanpa keraguan.
- **Sebagai Pembeli Cincin:** Saya ingin memilih ukuran cincin (*ring size*) yang tepat dan membaca panduan konversi ukuran internasional (US, HK, EU) langsung di tempat tanpa meninggalkan halaman.
- **Sebagai Klien Bernilai Tinggi (*HNW Client*):** Saya ingin membuat janji temu untuk melihat karya ini langsung di butik fisik atau mendiskusikan modifikasi kustom melalui tombol konsultasi pribadi.
- **Sebagai Pengguna Terdaftar:** Saya ingin menyimpan karya ini ke dalam daftar keinginan (*wishlist*) pribadi untuk ditinjau kembali di kemudian hari.

---

## 3. Aturan Bisnis (Business Rules)

1. **Integritas Sertifikasi Gemologi:**
   - Setiap karya yang memiliki berlian bernilai signifikan wajib mencantumkan laboratorium penerbit (`GIA` atau `IGI`) dan nomor sertifikat (`certificate_number`).
   - Tautan verifikasi sertifikat wajib terhubung langsung ke URL verifikasi publik resmi laboratorium terkait.
   - Tipografi nomor sertifikat wajib menggunakan font monospace `JetBrains Mono` untuk keterbacaan teknis yang presisi.
2. **Pilihan Varian & Ukuran:**
   - Jika karya adalah cincin (`category = rings`), opsi ukuran cincin (*ring size*) wajib dipilih sebelum tombol "Add to Bag" aktif.
   - Layanan pengubahan ukuran gratis pertama kali (*complimentary resizing*) dicantumkan sebagai bagian dari garansi layanan purnajual.
3. **Model Transaksi Hibrida (Hybrid Commerce Path):**
   - **Karya Siap Kirim (`in_stock`):** Menampilkan tombol "Add to Bag" (primer) dan "Book Boutique Viewing" (sekunder).
   - **Karya Dibuat Berdasarkan Pesanan (`made_to_order`):** Menampilkan tombol "Commission Piece" dengan estimasi pengerjaan (3-4 minggu) dan "Request Consultation".
   - **Karya Eksklusif/Terpesan (`reserved` / `sold_out`):** Menonaktifkan tombol "Add to Bag" dan mengubah CTA menjadi "Inquire on Similar Piece" yang membuka modal konsultasi concierge.
4. **Validasi Harga Dinamis:**
   - Harga yang ditampilkan pada antarmuka pengguna (`price_idr_minor` / `price_usd_minor`) bersifat read-only.
   - Ketika item ditambahkan ke keranjang belanja, server wajib memvalidasi ulang harga terkini dari basis data dan menolak harga yang dimanipulasi oleh client.

---

## 4. Integrasi Data (DATABASE.md)

Halaman ini memuat relasi data yang kaya:

- `pieces`: Data utama karya (`id`, `slug`, `sku`, `name`, `description`, `metal`, `setting_type`, `price_idr_minor`, `price_usd_minor`, `availability`, `is_published`).
- `diamonds`: Spesifikasi batu permata (`origin`, `carat`, `cut`, `color_grade`, `clarity_grade`, `shape`, `grading_lab`, `certificate_number`, `certificate_url`).
- `piece_variants`: Varian stok dan ukuran fisik (`id`, `piece_id`, `ring_size`, `stock`, `sku_suffix`).
- `piece_images`: Seluruh dokumentasi fotografi produk (`image_url`, `alt_text`, `display_order`, `is_primary`).
- `categories` & `collections`: Informasi konteks kategori dan narasi koleksi.
- `wishlist_items`: Status apakah karya sudah tersimpan di wishlist pengguna yang sedang login.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Read Endpoint (RSC & API)

- **`GET /api/pieces/[slug]`**
  - **Status 200 OK Response:**
    ```json
    {
      "success": true,
      "data": {
        "piece": {
          "id": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
          "slug": "celestial-solitaire-diamond-ring",
          "sku": "BJ-RNG-001",
          "name": "Celestial Solitaire Diamond Ring",
          "description": "Hand-forged 950 platinum setting holding a peerless 1.25ct natural round brilliant diamond.",
          "metal": "platinum",
          "settingType": "Four-prong classic trellis",
          "priceIdrMinor": 18500000000,
          "priceUsdMinor": 1180000,
          "availability": "in_stock",
          "images": [
            { "url": "https://blob.bifrostjewellery.com/pieces/celestial-1.webp", "altText": "Celestial Solitaire front view", "isPrimary": true },
            { "url": "https://blob.bifrostjewellery.com/pieces/celestial-2.webp", "altText": "Celestial Solitaire profile view", "isPrimary": false }
          ],
          "diamonds": [
            {
              "origin": "natural",
              "carat": 1.25,
              "shape": "round",
              "cut": "excellent",
              "colorGrade": "E",
              "clarityGrade": "VVS1",
              "gradingLab": "GIA",
              "certificateNumber": "GIA-2218491023",
              "certificateUrl": "https://www.gia.edu/report-check?reportno=2218491023"
            }
          ],
          "variants": [
            { "id": "var-05", "ringSize": "5.0", "stock": 1 },
            { "id": "var-06", "ringSize": "6.0", "stock": 2 },
            { "id": "var-07", "ringSize": "7.0", "stock": 1 }
          ]
        }
      }
    }
    ```

### Server Action Mutasi

- **`addToCartAction(variantId: string, quantity: number)`**
  - Menyimpan item ke session cart (tamu) atau basis data `cart_lines` (user terdaftar).
- **`toggleWishlistAction(pieceId: string)`**
  - Menambah/menghapus karya dari wishlist pengguna.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `GalleryLightbox`: Galeri foto editorial dengan rasio 4:5 vertikal, thumbnail strip rapi, fitur zoom resolusi tinggi tanpa distorsi gambar, dan transisi halus berbasis Framer Motion.
- `CertificationBadge`: Kotak lencana tenang dengan garis rambut (`--ql-line`), menampilkan logo teks laboratorium (GIA / IGI), nomor sertifikat (JetBrains Mono), dan tautan eksternal "Verify Report ↗".
- `SpecificationsTable`: Tabel 4Cs dua kolom yang elegan:
  - Carat Weight: `1.25 ct`
  - Cut Grade: `Excellent`
  - Color Grade: `E (Colorless)`
  - Clarity Grade: `VVS1 (Very, Very Slightly Included)`
  - Metal: `950 Platinum`
- `RingSizeSelector`: Dropdown atau tombol segmen ukuran cincin, berdampingan dengan tautan modal interaktif "Size Guide & Measurement Chart".
- `SizeGuideModal`: Modal berisi tabel perbandingan lingkar jari (mm) vs standar US/HK/EU dan instruksi pengukuran mandiri.
- `DualActionCTA`:
  - Tombol Primer: `Button` varian `primary` ("Add to Bag" berlatar `--ql-ink` dengan teks putih).
  - Tombol Sekunder: `Button` varian `outline` ("Book Boutique Appointment" dengan border halus).
- `ServiceGuarantees`: Ikon minimalis Phosphor Icons yang menggarisbawahi: Pengiriman Berasuransi Penuh, Sertifikasi Asli Terverifikasi, dan Layanan Resizing Pertama Cuma-Cuma.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Mengakses `/pieces/[slug]` yang valid mengembalikan halaman lengkap dengan status 200 OK dan metadata SEO OpenGraph yang dinamis.
- [ ] Mengakses slug yang tidak terdaftar mengembalikan halaman 404 Not Found yang terdesain rapi (*not-found.tsx*).
- [ ] Nomor sertifikat GIA atau IGI terhubung ke URL verifikasi eksternal yang benar dan terbuka di tab baru (`target="_blank" rel="noopener noreferrer"`).
- [ ] Pada karya perhiasan kategori cincin, penambahan ke keranjang tidak dapat dilakukan sebelum ukuran cincin dipilih.
- [ ] Mengklik "Add to Bag" memperbarui badge jumlah keranjang di Navbar secara instan dan memunculkan drawer konfirmasi keranjang mini (*Cart Drawer*).
- [ ] Mengklik "Book Boutique Appointment" membuka formulir janji temu dengan karya perhiasan otomatis terpilih sebagai referensi kunjungan.
- [ ] Rasio kontras teks pada tabel spesifikasi dan tombol aksi memenuhi standar WCAG AA (minimal 4.5:1).

---

## 8. Dokumen Terkait

- [CART_CHECKOUT.md](./CART_CHECKOUT.md) — Alur keranjang dan proses pembayaran
- [APPOINTMENT.md](./APPOINTMENT.md) — Alur pemesanan janji temu butik
- [CONSULTATION.md](./CONSULTATION.md) — Formulir konsultasi kustom
- [UI_UX.md](../UI_UX.md) — Panduan desain token dan komponen antarmuka
