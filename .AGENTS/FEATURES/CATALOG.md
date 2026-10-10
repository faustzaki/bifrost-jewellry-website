# 💎 FEATURE — Katalog & Koleksi Perhiasan (CATALOG)

> **Status:** Planned  
> **Scope:** Public Pages (Frontend + Data Layer)  
> **Primary Goal:** Menyajikan etalase perhiasan eksklusif dengan tata letak editorial tenang (*quiet luxury*), navigasi kategori intuitif, penyaringan dasar, dan pemuatan halaman ultra-cepat melalui Incremental Static Regeneration (ISR).

---

## 1. Ringkasan (Overview)

Fitur Katalog memungkinkan calon pelanggan menjelajahi seluruh koleksi mahakarya perhiasan Bifrost Jewellery. Desain mengadopsi estetika editorial bertaraf galeri seni perhiasan dengan tipografi Cormorant Garamond, rasio gambar konsisten, serta whitespace lapang. Pengguna dapat menelusuri koleksi berdasarkan kategori (cincin, kalung, anting, gelang), koleksi tematik khusus, ketersediaan stok, dan kisaran harga.

Untuk menjaga performa Core Web Vitals (LCP < 2.5s) dan efisiensi query basis data, halaman katalog publik memanfaatkan Server-Side Rendering (SSR) yang dikombinasikan dengan ISR (revalidasi bertahap pada interval tertentu), serta prefetching cerdas pada tautan produk.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pengunjung Butik Online:** Saya ingin melihat daftar karya perhiasan dengan foto beresolusi tinggi, nama karya, jenis logam mulia, dan harga, agar saya dapat menemukan perhiasan yang memikat selera estetika saya.
- **Sebagai Kolektor Perhiasan:** Saya ingin menyaring katalog berdasarkan kategori tertentu (misal: *Engagement Rings*, *High Jewellery Necklaces*) dan koleksi tematik, agar navigasi penelusuran lebih fokus.
- **Sebagai Pembeli:** Saya ingin menyortir perhiasan berdasarkan harga terendah, harga tertinggi, atau karya rilis terbaru, agar dapat menyesuaikan dengan anggaran belanja saya.
- **Sebagai Pengguna Perangkat Seluler:** Saya ingin menjelajahi katalog dengan transisi mulus, grid responsif yang rapi, dan panel filter geser yang mudah dioperasikan dengan satu tangan.

---

## 3. Aturan Bisnis (Business Rules)

1. **Visibilitas Produk:** Hanya karya perhiasan dengan status `is_published = true` yang dapat ditampilkan pada katalog publik. Karya berstatus draft atau tersembunyi hanya dapat dilihat melalui sesi admin terotentikasi.
2. **Ketersediaan & Label Stok:**
   - `in_stock`: Ditampilkan dengan harga normal dan tombol aksi langsung.
   - `made_to_order`: Diberi label elegan "Made to Order" dengan perkiraan waktu pengerjaan 3-4 minggu.
   - `reserved`: Diberi label "Reserved" (sedang dalam reservasi butik), opsi konsultasi tetap terbuka.
   - `sold_out`: Ditampilkan di bagian bawah katalog dengan opsi "Request Similar Piece" via formulir konsultasi.
3. **Penyajian Mata Uang Ganda:**
   - Nilai harga disimpan dalam unit minor integer (`price_idr_minor` dan `price_usd_minor`).
   - Tampilan mata uang mengikuti preferensi aktif pengguna (default IDR untuk pengunjung domestik, USD untuk internasional).
   - Simbol mata uang diformat secara ketat: `IDR Rp 45.000.000` atau `USD $3,200`.
4. **Caching & Revalidasi:**
   - Halaman root `/collections` dan halaman per kategori `/collections/[slug]` menggunakan Next.js ISR dengan batas `revalidate = 3600` (1 jam).
   - On-demand revalidation dijalankan melalui Server Action admin setiap kali ada pembaruan data produk (`revalidatePath('/collections')`).

---

## 4. Integrasi Data (DATABASE.md)

Fitur ini berinteraksi langsung dengan tabel-tabel berikut:

- `pieces`: Entitas utama karya perhiasan (`id`, `slug`, `sku`, `name`, `description`, `category_id`, `collection_id`, `metal`, `setting_type`, `price_idr_minor`, `price_usd_minor`, `availability`, `is_published`, `is_featured`).
- `categories`: Pengelompokan jenis perhiasan (`id`, `slug`, `name`, `description`, `display_order`).
- `collections`: Pengelompokan tema kurasi (`id`, `slug`, `name`, `story`, `banner_image_url`, `is_active`).
- `piece_images`: Foto produk (`id`, `piece_id`, `image_url`, `alt_text`, `display_order`, `is_primary`).
- `diamonds`: Relasi batu berlian utama untuk menampilkan atribut ringkas (origin, carat, cut) pada kartu produk jika relevan.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Read Operations (Server Components & API Route)

- **`GET /api/pieces`**
  - **Query Parameters:**
    - `category`: Filter slug kategori (contoh: `rings`, `necklaces`).
    - `collection`: Filter slug koleksi (contoh: `aurora-2026`).
    - `metal`: Filter enum jenis logam (`gold_18k`, `gold_14k`, `platinum`, `rose_gold_18k`).
    - `availability`: Filter status (`in_stock`, `made_to_order`).
    - `sort`: Opsi pengurutan (`newest`, `price_asc`, `price_desc`).
    - `page`: Nomor halaman (default: `1`, limit: `12`).
  - **Response 200 OK:**
    ```json
    {
      "success": true,
      "data": {
        "pieces": [
          {
            "id": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
            "slug": "solitaire-celestial-ring",
            "sku": "BJ-RNG-001",
            "name": "Celestial Solitaire Diamond Ring",
            "metal": "platinum",
            "priceIdrMinor": 4500000000,
            "priceUsdMinor": 285000,
            "availability": "in_stock",
            "primaryImage": "https://blob.bifrostjewellery.com/pieces/celestial-solitaire-primary.webp",
            "category": { "name": "Rings", "slug": "rings" }
          }
        ],
        "pagination": {
          "currentPage": 1,
          "totalPages": 4,
          "totalItems": 48,
          "hasNext": true
        }
      }
    }
    ```

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `Navbar`: Varian solid atau transparan dengan tautan navigasi kategori aktif.
- `CollectionHero`: Banner visual minimalis dengan headline Cormorant Garamond dan pengantar singkat narasi kurasi.
- `FilterPanel`: Panel filter samping (desktop) atau drawer geser halus (mobile) yang memungkinkan penyaringan kategori, material logam, dan kisaran harga.
- `ProductCard`:
  - Rasio gambar 4:5 vertikal mewah dengan background `--ql-surface`.
  - Efek hover halus: transisi ke foto sekunder (*angle detail*) dalam durasi 400ms.
  - Teks nama karya (Inter Regular / Cormorant 18px), logam mulia, dan label harga diformat rapi.
  - Badge ketersediaan elegan jika *Made to Order* atau *Reserved*.
- `PaginationControls`: Navigasi halaman angka minimalis tanpa panah dekoratif mencolok.
- `EmptyState`: Tampilan bersih jika hasil filter nihil, dilengkapi tombol reset filter "Clear all filters".

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Halaman `/collections` menampilkan karya perhiasan terbit (`is_published = true`) dalam layout grid 3 kolom (desktop) dan 1-2 kolom (mobile).
- [ ] Filter berdasarkan kategori, material logam, dan sorting harga berfungsi secara instan melalui manipulasi URL Search Params tanpa reload penuh halaman.
- [ ] Foto utama produk di-render menggunakan `next/image` dengan atribut `sizes`, format modern (WebP/AVIF), dan `priority` hanya pada 3 item pertama baris atas.
- [ ] Nilai harga ditampilkan secara akurat sesuai mata uang yang dipilih pengguna (IDR atau USD) menggunakan helper `formatMoney`.
- [ ] Indikator visual *loading state* (skeleton placeholder berwarna `--ql-line`) tampil halus saat transisi halaman katalog berlangsung.
- [ ] Revalidasi ISR bekerja tepat waktu dan cache diperbarui segera saat Server Action pembaruan produk dipanggil.

---

## 8. Dokumen Terkait

- [DIAMOND_SEARCH.md](./DIAMOND_SEARCH.md) — Penyaringan berlian spesifik 4Cs
- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Halaman spesifikasi detail perhiasan
- [CATALOG_QUERY_PATTERN.md](../PATTERNS/CATALOG_QUERY_PATTERN.md) — Pola query builder Prisma dan indexing katalog
- [UI_UX.md](../UI_UX.md) — Desain token warna, tipografi, dan tata letak
