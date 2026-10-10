# 📖 FEATURE — Halaman Editorial & Panduan (EDITORIAL)

> **Status:** Planned  
> **Scope:** Brand Editorial, Craftsmanship Journal, Care & Size Guides  
> **Primary Goal:** Membangun narasi warisan seni (*brand heritage*), edukasi gemologi, panduan perawatan perhiasan (*care guide*), dan panduan pengukuran presisi (*size guide*) melalui tata letak editorial majalah seni bertaraf tinggi.

---

## 1. Ringkasan (Overview)

Sebagai merek perhiasan *quiet luxury*, Bifrost Jewellery tidak hanya menjual produk fisik, tetapi juga mentransmisikan nilai-nilai keindahan abadi, etika penambangan yang bertanggung jawab, serta dedikasi keterampilan tangan para pengrajin ulung (*master jewellers*).

Modul Editorial menghadirkan serangkaian halaman naratif kaya estetika:
1. **The Journal / Craftsmanship Story (`/journal`):** Esai mendalam mengenai proses peleburan platinum, teknik tatahan mikro (*micro-pavé setting*), pemilihan berlian, serta wawancara dengan perancang karya.
2. **About the Maison (`/about`):** Filosofi desain, komitmen transparansi dual-origin (natural vs lab-grown), dan integritas sumber material ramah lingkungan.
3. **Jewellery Care Guide (`/care-guide`):** Petunjuk ilmiah dan praktis mengenai perawatan harian logam mulia dan batu permata agar kilaunya bertahan lintas generasi.
4. **Ring & Fine Jewellery Size Guide (`/size-guide`):** Panduan konversi ukuran internasional, metode pengukuran lingkar jemari mandiri, dan penjelasan layanan *complimentary resizing*.

Seluruh halaman ini di-render secara statis (SSG) dengan visual penuh keanggunan, tipografi Cormorant Garamond, dan performa pemuatan super cepat.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Penikmat Seni Perhiasan:** Saya ingin membaca esai tentang proses pembuatan perhiasan Bifrost dan filosofi pemilihan batu permata agar saya memahami nilai keahlian di balik karya yang saya beli.
- **Sebagai Pemilik Perhiasan:** Saya ingin membaca panduan perawatan yang benar mengenai cara membersihkan cincin platinum dan berlian di rumah tanpa merusak struktur rangkanya.
- **Sebagai Calon Pembeli Cincin Hadiah:** Saya ingin mempelajari panduan ukuran cincin yang komprehensif agar saya dapat memperkirakan ukuran jari pasangan saya secara akurat.
- **Sebagai Pembaca Web:** Saya ingin menikmati pengalaman membaca yang nyaman dengan tipografi serif klasik yang mudah dibaca, tata letak majalah tanpa gangguan iklan, dan pemuatan gambar yang halus.

---

## 3. Aturan Bisnis (Business Rules)

1. **Integritas Narasi & Tone of Voice:**
   - Semua naskah editorial wajib mematuhi panduan suara *quiet luxury*: bernada tenang, berwibawa, menghargai seni kriya, dan bebas dari bahasa pemasaran agresif (dilarang menggunakan kata seru, diskon, atau promosi berteriak).
2. **Kepatuhan Penjelasan Teknis:**
   - Panduan gemologi dan ukuran wajib akurat secara ilmiah (mengacu pada standar metrik ISO untuk lingkar jari dan skala ketahanan mineral Mohs untuk batu permata).
3. **Strategi Caching Statis (SSG / Static Site Generation):**
   - Halaman editorial bersifat statis dan di-generate saat proses build (`output: 'export'` atau SSG route).
   - Revalidasi hanya dipicu saat terjadi pembaruan artikel oleh tim konten.
4. **SEO & Skema Metadata:**
   - Halaman artikel jurnal wajib menyertakan script JSON-LD `Article` / `BlogPosting`.
   - Menghasilkan OpenGraph image dan meta tag terstruktur untuk keterbagian di media sosial.

---

## 4. Integrasi Data (DATABASE.md)

- Pada fase rilis awal, konten editorial, panduan ukuran, dan petunjuk perawatan dikelola sebagai dokumen Markdown / MDX statis berbasis file lokal atau tabel konten terkelola:
  - `articles` (opsional untuk fase selanjutnya): `id`, `slug`, `title`, `excerpt`, `body_mdx`, `hero_image_url`, `author`, `published_at`, `is_published`.
- Komponen panduan ukuran cincin (`SizeGuideTable`) menyajikan data konversi statis:
  - Diameter (mm), Lingkar Jari / Circumference (mm), Ukuran US (3.0 - 12.0), Ukuran HK (6 - 25), Ukuran EU (44 - 68).

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

- Halaman editorial diakses melalui rute publik statis:
  - `GET /about`
  - `GET /journal`
  - `GET /journal/[slug]`
  - `GET /care-guide`
  - `GET /size-guide`
- Data disajikan melalui React Server Components tanpa endpoint mutasi publik.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `EditorialHero`: Tampilan judul utama majalah dengan tipografi Cormorant Garamond 48px/64px, pengantar naratif singkat, dan fotografi panorama studio berbingkai halus.
- `StoryBlock`: Blok esai dua kolom yang memadukan kutipan pengrajin (*pull quote* beraksen `--ql-gold-deep`) dan foto detail makro proses kerja atelier.
- `CareGuideAccordion`: Tata letak kartu lipat akordeon minimalis yang menguraikan tips pembersihan harian, penyimpanan perhiasan yang aman, dan jadwal inspeksi berkala.
- `SizeComparisonTable`: Tabel interaktif responsif yang memetakan lingkar jari dalam milimeter ke ukuran standar US, HK, dan EU dengan penyorot baris (*row highlight*) saat kursor diarahkan.
- `PrintableMeasurementModal`: Tautan unduhan berkas panduan cetak PDF skala 100% (*printable ring sizer ruler*) yang siap dicetak di rumah.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Seluruh halaman editorial (`/about`, `/journal`, `/care-guide`, `/size-guide`) dapat diakses secara publik dan menghasilkan skor Core Web Vitals LCP < 2.0 detik.
- [ ] Tabel konversi ukuran cincin merender data metrik milimeter dan ukuran internasional (US, HK, EU) dengan benar dan mudah dibaca pada perangkat mobile.
- [ ] Tautan unduh panduan ukuran cetak menyediakan berkas PDF beresolusi tinggi dengan garis skala kalibrasi kartu kredit untuk memverifikasi skala cetak 100%.
- [ ] Seluruh foto editorial dioptimasi dengan `next/image`, format WebP/AVIF, dan rasio aspek responsif tanpa pergeseran tata letak (*CLS = 0*).
- [ ] Konten artikel jurnal menyertakan metadata JSON-LD `Article` yang lolos uji Google Rich Results.

---

## 8. Dokumen Terkait

- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Modal panduan ukuran cincin yang bersumber dari spesifikasi ini
- [UI_UX.md](../UI_UX.md) — Prinsip tipografi Cormorant Garamond dan tone of voice
