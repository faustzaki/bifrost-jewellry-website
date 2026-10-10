# 💎 FEATURE — Pencarian & Filter Berlian 4Cs (DIAMOND_SEARCH)

> **Status:** Planned  
> **Scope:** Public Pages & Search Experience  
> **Primary Goal:** Memberikan pengalaman interaktif bagi kolektor dan pembeli perhiasan untuk mencari dan memfilter berlian bersertifikat berdasarkan asal usul (*origin*), bentuk (*shape*), parameter 4Cs (Carat, Cut, Color, Clarity), rentang harga, dan laboratorium gemologi resmi.

---

## 1. Ringkasan (Overview)

Bifrost Jewellery memegang prinsip transparansi penuh dan integritas gemologi tinggi. Fitur Pencarian Berlian (*Diamond Search*) menyediakan mesin penelusuran berkecepatan tinggi yang memungkinkan pengguna menyaring inventaris berlian secara presisi. Baik pelanggan yang mencari kemurnian berlian tambang alami (*natural diamond*) maupun ketelitian berlian hasil laboratorium berteknologi mutakhir (*lab-grown diamond*), semua parameter disajikan secara jujur dengan sertifikasi resmi (GIA atau IGI).

Antarmuka pencarian didesain dengan visual tenang (*quiet luxury*): slider rentang karat yang mulus, selektor bentuk ikonik, serta matriks warna dan kejernihan yang jelas tanpa kebingungan teknis.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pembeli Cincin Tunangan:** Saya ingin memilih asal usul berlian (*natural* vs *lab-grown*) secara eksplisit, agar saya mendapatkan transparansi asal batu sesuai filosofi dan alokasi anggaran saya.
- **Sebagai Penikmat Desain Klasik:** Saya ingin memilih bentuk berlian tertentu (Round, Oval, Emerald, Pear, Princess, Marquise, Cushion) dan melihat siluet proporsi bentuknya, agar saya bisa membayangkan estetika saat terpasang pada cincin.
- **Sebagai Kolektor Cermat:** Saya ingin memfilter berlian berdasarkan parameter 4Cs standar internasional (Cut: Excellent/Very Good; Color: D-J; Clarity: FL-SI1; Carat: rentang desimal), agar saya menemukan batu dengan spesifikasi tepat.
- **Sebagai Pengguna Anggaran Terukur:** Saya ingin menentukan batas minimum dan maksimum harga dalam mata uang pilihan saya (IDR atau USD) dan menyortir hasil pencarian secara instan.
- **Sebagai Pengunjung Mobile:** Saya ingin melihat jumlah berlian yang cocok secara real-time saat menggeser slider filter tanpa harus mengalami reload halaman.

---

## 3. Aturan Bisnis (Business Rules)

1. **Dua Kategori Asal Usul Berlian (Dual Origin):**
   - Setiap entitas berlian wajib memiliki nilai enum `origin` yang tegas: `natural` atau `lab_grown`.
   - Tidak diperkenankan menyamarkan atau mencampuradukkan kedua kategori asal ini dalam satu tampilan tanpa label penjelas.
2. **Kepatuhan Parameter 4Cs:**
   - **Carat (Karat):** Disimpan dalam tipe angka desimal berpresisi 2 digit (`numeric(5,2)`), rentang filter 0.30 ct hingga 10.00+ ct.
   - **Cut (Potongan):** Standar internasional (`excellent`, `very_good`, `good`, `fair`).
   - **Color (Warna):** Skala warna D (Colorless sempurna) hingga J (Near Colorless).
   - **Clarity (Kejernihan):** Skala kejernihan FL/IF, VVS1, VVS2, VS1, VS2, SI1.
3. **Mandat Sertifikasi Resmi:**
   - Setiap batu berlian wajib mencantumkan laboratorium penerbit sertifikat: `GIA` (*Gemological Institute of America*) atau `IGI` (*International Gemological Institute*).
   - Nomor laporan sertifikat (`certificate_number`) wajib unik dan tercatat dalam sistem.
4. **Sinkronisasi Ketersediaan:**
   - Berlian yang sudah terpasang pada karya perhiasan yang terjual (`sold_out`) atau dipesan (*reserved*) otomatis disembunyikan dari hasil filter aktif.
5. **Debounce & URL Sync:**
   - Input slider (karat dan harga) harus menerapkan debounce 300ms sebelum memicu query baru ke server.
   - Setiap kondisi filter harus tersimpan dalam URL Search Params agar tautan hasil pencarian dapat dibagikan (*shareable URL*).

---

## 4. Integrasi Data (DATABASE.md)

Fitur ini membaca dan memproses data dari tabel:

- `diamonds`: Data teknis gemologi (`id`, `piece_id`, `origin`, `carat`, `cut`, `color_grade`, `clarity_grade`, `shape`, `grading_lab`, `certificate_number`, `certificate_url`).
- `pieces`: Relasi perhiasan pengikat batu (`id`, `name`, `slug`, `price_idr_minor`, `price_usd_minor`, `availability`, `is_published`).
- `piece_images`: Foto detail berlian dan visual 360-derajat jika tersedia.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Query Endpoint

- **`GET /api/pieces` (Mode Diamond Filter)**
  - **Query Parameters:**
    - `origin`: `natural` | `lab_grown` | `all`
    - `shape`: `round,oval,emerald,pear,princess,marquise,cushion` (comma-separated)
    - `carat_min`: angka desimal (contoh: `0.70`)
    - `carat_max`: angka desimal (contoh: `2.50`)
    - `cut`: `excellent,very_good`
    - `color`: `D,E,F,G`
    - `clarity`: `VVS1,VVS2,VS1`
    - `price_min`: integer minor unit
    - `price_max`: integer minor unit
    - `grading_lab`: `GIA,IGI`
    - `sort`: `carat_desc`, `carat_asc`, `price_asc`, `price_desc`, `newest`
    - `page`: nomor halaman (default: `1`, limit: `15`)
  - **Response 200 OK:**
    ```json
    {
      "success": true,
      "data": {
        "diamonds": [
          {
            "id": "d1e2f3a4-b5c6-7d8e-9f0a-1b2c3d4e5f6a",
            "origin": "natural",
            "carat": 1.25,
            "shape": "round",
            "cut": "excellent",
            "colorGrade": "E",
            "clarityGrade": "VVS1",
            "gradingLab": "GIA",
            "certificateNumber": "GIA-2218491023",
            "certificateUrl": "https://www.gia.edu/report-check?reportno=2218491023",
            "piece": {
              "id": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
              "slug": "celestial-solitaire-125ct",
              "name": "Celestial Solitaire 1.25ct",
              "priceIdrMinor": 18500000000,
              "priceUsdMinor": 1180000
            }
          }
        ],
        "meta": {
          "totalResults": 38,
          "minPriceIdrMinor": 4500000000,
          "maxPriceIdrMinor": 45000000000,
          "minCarat": 0.50,
          "maxCarat": 4.50
        }
      }
    }
    ```

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `OriginSegmentedControl`: Tombol alih gaya editorial: "Natural Diamonds" dan "Lab-Grown Diamonds" dengan indikator garis emas tenang (`--ql-gold`).
- `ShapeSelector`: Kisi selektor bentuk berlian dengan ilustrasi garis geometris minimalis (vektor SVG hairline), menampilkan bentuk aktif dengan border aksen.
- `DualRangeSlider`: Komponen slider kustom untuk rentang karat dan rentang harga dengan angka indikator akurat tanpa melompat.
- `GradePillGroup`: Kumpulan tombol pill multi-pilih untuk *Color Grades* (D-J) dan *Clarity Grades* (FL hingga SI1) dengan feedback visual kontras AA.
- `DiamondResultsTable` (Opsi Desktop): Tampilan tabel gemologi editorial dengan tipografi JetBrains Mono untuk angka sertifikat dan ukuran desimal karat.
- `DiamondCardGrid` (Opsi Mobile/Visual): Tampilan kartu vertikal dengan foto batu makro beresolusi tajam.
- `SearchEmptyState`: Notifikasi elegan ketika kombinasi filter terlalu sempit, menyediakan saran pelonggaran filter atau tautan langsung ke formulir *Bespoke Sourcing Request*.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Memilih tab "Natural" hanya mengembalikan data berlian dengan `origin = natural`, dan tab "Lab-Grown" hanya mengembalikan `origin = lab_grown`.
- [ ] Multi-seleksi bentuk (*shape*) bekerja secara or-inclusive (misal: memilih Round dan Oval menampilkan kedua bentuk tersebut).
- [ ] Filter slider rentang karat membatasi hasil pencarian secara presisi matematis sesuai interval `[carat_min, carat_max]`.
- [ ] Pengurutan (*sorting*) berdasarkan karat atau harga berfungsi benar dan mempertahankan parameter filter aktif.
- [ ] Perubahan filter secara otomatis memperbarui URL query string di peramban tanpa memicu scroll jump atau refresh halaman penuh.
- [ ] State hasil kosong (*Empty State*) menampilkan pesan informatif dan tombol "Reset Filters" yang mengembalikan semua parameter ke nilai default.

---

## 8. Dokumen Terkait

- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Integrasi tampilan batu pada halaman karya jadi
- [CATALOG_QUERY_PATTERN.md](../PATTERNS/CATALOG_QUERY_PATTERN.md) — Implementasi filter builder dinamis Prisma
- [GLOSSARY.md](../GLOSSARY.md) — Definisi standar parameter 4Cs dan terminologi gemologi
