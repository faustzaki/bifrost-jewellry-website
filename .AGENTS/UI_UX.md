# UI_UX.md — Sistem Desain Quiet Luxury & Standar Editorial

> **Tujuan:** Dokumen ini adalah sumber kebenaran visual (*visual source of truth*) untuk proyek Bifrost Jewellery. Dokumen ini menetapkan prinsip estetika quiet luxury, spesifikasi token warna terverifikasi WCAG AA, skala tipografi presisi, aturan spacing tata letak, perilaku komponen antarmuka, motion halus, serta standar copywriting editorial.

---

## 1. Prinsip Desain Quiet Luxury (Design Principles)

1. **Restraint (Kesederhanaan Elegan Tanpa Riuh):** Kemewahan sejati tidak perlu berteriak. Kami menolak elemen grafis yang ramai, gradien warna-warni yang mencolok, dan ornamen dekoratif yang tidak memiliki tujuan esensial. Setiap elemen visual harus memiliki alasan keberadaan yang kuat.
2. **Ruang Bernafas sebagai Struktur (Whitespace as Architecture):** Ruang kosong (*negative space*) bukanlah area kosong yang belum terisi, melainkan elemen arsitektural utama yang memberikan ketenangan visual, ruang napas, dan membimbing mata pengunjung menuju karya seni perhiasan.
3. **Fotografi Perhiasan sebagai Pusat Narasi (Jewellery First):** Citra fotografi beresolusi tinggi dengan pencahayaan alami, bayangan lembut, dan detail tekstur logam yang tajam menjadi pembawa cerita utama. Antarmuka bertindak sebagai galeri kurasi yang hening.
4. **Satu Hero per Tampilan (Singular Focus):** Setiap viewport hanya boleh memiliki satu fokus visual utama. Hindari persaingan hierarki antara banner promosi, badge berkedip, dan tombol aksi yang bertumpuk.
5. **Gerakan yang Tenang & Presisi (Quiet Motion):** Interaksi gerak bersifat organik, halus, dan hening. Kami tidak menggunakan animasi agresif yang mengagetkan pengguna.

---

## 2. Token Warna Semantik (Semantic Color Tokens)

Tabel di bawah ini menetapkan token CSS resmi yang telah diuji rasio kontrasnya menggunakan standar WCAG 2.1 AA/AAA terhadap latar belakang kanvas. **Dilarang menambahkan warna baru di luar tabel ini.**

| Token CSS | Nilai Hex | Peruntukan Antarmuka | Rasio Kontras & Uji Kepatuhan WCAG |
|---|---|---|---|
| `--ql-canvas` | `#FAF8F5` | Latar belakang halaman utama (*warm off-white canvas*) | Latar dasar seluruh halaman |
| `--ql-surface` | `#FFFFFF` | Latar belakang kartu perhiasan, sheet drawer, dan modal popover | 1.05:1 vs canvas (kontras elevasi halus) |
| `--ql-ink` | `#1A1A1A` | Teks utama, judul editorial, dan label navigasi primer | **16.42:1 vs canvas ✅ Lulus WCAG AAA** |
| `--ql-ink-muted` | `#6B6259` | Teks pendukung, deskripsi produk, spesifikasi teknis | **5.63:1 vs canvas ✅ Lulus WCAG AA** |
| `--ql-line` | `#E7E2DA` | Garis pembatas tipis (*hairline border* 1px) dan garis tabel | Elemen non-teks struktural |
| `--ql-gold` | `#9C7C3F` | Ornamen aksen emas dan **teks display besar (≥ 24px)** | 3.69:1 vs canvas (**Hanya untuk teks besar ≥ 24px**); 7.26:1 pada ink |
| `--ql-gold-deep` | `#7C6236` | Teks aksen emas untuk **ukuran normal & kecil (< 24px)** | **5.42:1 vs canvas ✅ Lulus WCAG AA** |
| `--ql-ink-surface` | `#1A1A1A` | Blok editorial gelap pekat, section narasi dramatis, dan footer | Teks putih di atasnya **16.42:1 ✅ WCAG AAA** |
| `--ql-success` | `#15803D` | Indikator status sukses, pembayaran lunas, ketersediaan stok | Status positif |
| `--ql-error` | `#B3261E` | Pesan error validasi formulir dan indikator stok habis | Status peringatan kegagalan |

### Aturan Emas Penggunaan Warna (Hard Color Rules)
- **Aturan Emas 1:** DILARANG menggunakan token `--ql-gold` pada teks berukuran normal (< 24px) di atas latar `--ql-canvas`. Untuk teks aksen emas ukuran normal/kecil, **WAJIB** menggunakan token `--ql-gold-deep` agar memenuhi batas kontras AA (5.42:1).
- **Aturan Emas 2:** DILARANG menggunakan warna hitam murni `#000000`. Seluruh teks gelap dan permukaan gelap wajib menggunakan `--ql-ink` / `--ql-ink-surface` (`#1A1A1A`).
- **Aturan Emas 3:** DILARANG menggunakan gradien warna dekoratif (seperti gradien pelangi atau gradien emas mencolok). Warna emas hadir sebagai aksen murni (*solid gold tone*).

---

## 3. Tipografi & Skala Tipe Presisi (Typography Scale)

Sistem tipografi menggunakan perpaduan antara keanggunan klasik dan keterbacaan modern:
1. **Display & Headlines:** `Cormorant Garamond` (Weights: 400 Regular, 500 Medium). Menghadirkan wibawa editorial perhiasan adibusana.
2. **Body & Interface:** `Inter` (Weights: 400 Regular, 500 Medium, 600 Semi-bold). Memastikan keterbacaan prima pada form, navigasi, dan informasi mikro.
3. **Monospace & Metadata:** `JetBrains Mono` (Weights: 400 Regular, 500 Medium). Digunakan khusus untuk nomor sertifikat GIA/IGI, nomor SKU, dan kode pesanan.

### Skala Tipografi Terkunci (Type Scale Table)

| Tingkat (Step) | Ukuran (px / rem) | Font Family | Tracking & Line Height | Penggunaan Resmi |
|---|---|---|---|---|
| `4xl` | 88px / 5.5rem | Cormorant Garamond | Tracking `-0.02em`, `leading-none` | Hero Display utama beranda desktop |
| `3xl` | 64px / 4rem | Cormorant Garamond | Tracking `-0.02em`, `leading-[1.1]` | Headline koleksi khusus & editorial besar |
| `2xl` | 48px / 3rem | Cormorant Garamond | Tracking `-0.01em`, `leading-tight` | Judul halaman (H1), nama perhiasan di PDP |
| `xl` | 32px / 2rem | Cormorant Garamond | Tracking normal, `leading-tight` | Sub-judul section (H2), nama koleksi |
| `lg` | 24px / 1.5rem | Cormorant Garamond | Tracking normal, `leading-snug` | Judul kartu perhiasan (H3), judul modal |
| `md` | 18px / 1.125rem | Inter | Tracking normal, `leading-relaxed` | Paragraf pembuka editorial (*lead text*) |
| `base` | 16px / 1rem | Inter | Tracking normal, `leading-normal` | Body text standar, isi artikel, opsi formulir |
| `sm` | 14px / 0.875rem | Inter | Tracking `+0.01em`, `leading-normal` | Label input formulir, tombol navigasi, harga |
| `xs` | 12px / 0.75rem | Inter / JetBrains Mono | Tracking `+0.12em` (uppercase), `leading-tight` | Badge status, kategori, nomor sertifikat GIA |

> ⚠️ **Peringatan Ketat:** Dilarang menggunakan ukuran font sembarangan (*arbitrary font-size*) di luar 9 tingkatan resmi di atas.

---

## 4. Spacing, Grid, & Layout Editorial

- **Container Maksimum:** `max-w-[1280px]` dengan perataan tengah otomatis (`mx-auto`).
- **Padding Horizontal Layar:**
  - Mobile (< 768px): `px-5` (20px).
  - Desktop (≥ 1024px): `px-12` (48px).
- **Sistem Grid Desktop:** Grid 12 kolom dengan jarak antar kolom (*gutter*) `gap-6` atau `gap-8`.
- **Ritme Spacing Vertikal Antar Section:**
  - Mobile: `py-16` (64px).
  - Desktop: `py-24 lg:py-32` (96px hingga 128px) untuk memberikan jeda visual yang luas.
- **Sudut Lengkung (Border Radius):**
  - Kartu Produk & Kontainer Editorial: `rounded-[2px]` (nyaris siku-siku tajam berkarakter arsitektural).
  - Tombol Aksi (Buttons) & Badge: `rounded-full` (bentuk kapsul halus/pill).
- **Garis Pembatas (Borders):** Garis pembatas selalu tipis 1px (`border border-ql-line`). Dilarang menggunakan bayangan kartu tebal (*no heavy drop shadows*).

---

## 5. Pustaka Komponen Antarmuka (Component Library)

| Komponen | Varian & State | Karakteristik Visual Quiet Luxury |
|---|---|---|
| **Button** | `primary` (latar ink, teks putih)<br>`outline` (border line, teks ink)<br>`quiet` (tanpa border, underline halus saat hover) | Kapsul `rounded-full`, padding presisi `px-8 py-3.5`, transisi warna halus, teks `text-sm font-medium`. |
| **Input / Select** | `default`, `focus`, `error` | Latar transparan atau surface, border bawah atau border 1px halus, fokus ring emas dalam `ring-1 ring-ql-gold-deep`. |
| **QuantityStepper** | `compact` | Tombol minus/plus minimalis tanpa panah default browser, teks angka diapit rapi. |
| **ProductCard** | `grid-item`, `featured` | Rasio foto 3:4 atau 4:5, hover zoom mikro (1.02x scale), nama perhiasan Cormorant Garamond, harga minor unit rapi. |
| **CollectionCard** | `editorial-banner` | Rasio lebar editorial, tipografi serif anggun di atas overlay transparan hening. |
| **FilterPanel** | `drawer-mobile`, `sidebar-desktop` | Opsi filter 4Cs berbasis pill tombol minimalis, slider rentang karat presisi. |
| **Gallery / Lightbox** | `multi-angle`, `fullscreen-zoom` | Navigasi thumbnail halus, latar belakang gelap lembut, transisi cross-fade tanpa sentakan. |
| **SizeGuide** | `modal-table` | Tabel konversi ukuran cincin standar US/HK/Diameter internal, petunjuk cara mengukur mandiri. |
| **Badge** | `origin` (natural/lab), `availability`, `lab` (GIA/IGI) | Ukuran ringkas `text-xs uppercase tracking-widest`, border tipis, warna latar netral. |
| **Modal / Dialog** | `center-editorial` | Backdrop blur halus (`backdrop-blur-sm bg-ql-ink/30`), kontainer surface dengan border 1px `border-ql-line`. |
| **Toast** | `success`, `error`, `info` | Notifikasi ringkas melayang di kanan bawah, tanpa ikon mencolok yang mengganggu. |
| **Accordion (FAQ)** | `single-open` | Garis pemisah hairline, animasi buka-tutup halus, ikon plus tipis yang berputar lembut menjadi minus. |
| **Navbar** | `transparent-over-hero`<br>`solid-scrolled` | Navigasi hening di atas kanvas, sticky dengan transisi ke solid surface saat di-scroll. |
| **Footer** | `editorial-dark` | Memanfaatkan latar `--ql-ink-surface` (`#1A1A1A`), teks `--ql-canvas`, tautan etika batu mulia dan layanan butik. |
| **AppointmentPicker**| `interactive-calendar` | Kalender pemilihan tanggal elegan, pemilihan slot jam siang/sore, form data tamu terintegrasi. |
| **Tabs** | `underline-editorial` | Garis aktif emas tipis bergerak mengikuti tab terpilih. |

---

## 6. Motion & Animasi Halus (Motion System)

- **Durasi Standar:** 300ms hingga 500ms untuk transisi tampilan dan modal.
- **Timing Function (Easing):**
  ```css
  cubic-bezier(0.22, 1, 0.36, 1) /* Lembut dan berbobot tanpa memantul */
  ```
- **Smooth Scroll (Lenis):** Seluruh pergerakan scroll halaman menggunakan inertia scrolling halus via pustaka Lenis.
- **Kepatuhan Terhadap Aksesibilitas Motion:**
  - Wajib menghormati preferensi pengguna `prefers-reduced-motion: reduce`.
  - Jika preferensi aktif, seluruh durasi transisi dipangkas menjadi 0ms (instan).
- **Larangan Animasi Mutlak:**
  - Dilarang menggunakan parallax agresif multi-layer.
  - Dilarang melakukan scroll-hijacking (mengubah kecepatan scroll mouse alami pengguna secara paksa).
  - Dilarang menggunakan teks berjalan (*marquee*) berkedip.
  - Dilarang menggunakan carousel otomatis yang berputar sendiri tanpa kontrol pengguna.

---

## 7. Checklist Aksesibilitas (Accessibility / a11y)

- [ ] Seluruh pasangan warna teks dan latar belakang telah lolos verifikasi rasio kontras WCAG AA (minimal 4.5:1 untuk teks normal, minimal 3:1 untuk teks besar).
- [ ] Seluruh kontrol interaktif dapat dioperasikan penuh menggunakan keyboard melalui tombol `Tab`, `Enter`, dan `Space`.
- [ ] Indikator fokus terlihat jelas melalui ring semantik (`focus-visible:ring-1 focus-visible:ring-ql-gold-deep`).
- [ ] Seluruh gambar perhiasan memiliki atribut `alt` deskriptif yang mencerminkan nama perhiasan dan jenis logam.
- [ ] Elemen form memiliki elemen `<label>` yang terhubung secara semantik via atribut `htmlFor`.
- [ ] Atribut ARIA disematkan secara tepat pada komponen dinamis (`aria-expanded`, `aria-haspopup`, `aria-selected`).

---

## 8. Standar Tone of Voice & Copywriting Editorial

Copywriting antarmuka publik wajib mencerminkan citra rumah perhiasan mewah internasional:
1. **Dilarang Tanda Seru (!):** Tanda seru dilarang digunakan di seluruh headline, subheadline, maupun deskripsi produk. Kemewahan sejati berbicara dengan nada tenang dan pasti.
2. **Dilarang Kata-Kata Ritel Murahan:**
   - Dilarang menggunakan kata: *"Sale"*, *"Diskon"*, *"Promo"*, *"Deal"*, *"Hemat"*, *"Spesial Hari Ini"*, *"Murah"*.
   - Gantilah dengan pendekatan kurasi: *"The Bridal Edit"*, *"Complimentary Ring Resizing"*, *"Curated Creations"*.
3. **Dilarang Huruf Kapital Berteriak (No ALL-CAPS Shouting):** Seluruh judul utama ditulis dalam format **Sentence case** (contoh: *"The radiance of ethically sourced diamonds"*). Huruf kapital penuh HANYA diperbolehkan untuk label mikro 12px dengan tracking longgar `tracking-widest`.
4. **Fokus pada Provenance & Craftsmanship:** Deskripsikan keahlian tatah logam tangan, kejernihan asahan berlian, dan sejarah perhiasan. Jangan gunakan teknik psikologi urgensi (*false urgency*).
5. **Transparansi Sertifikasi Mutlak:** Setiap klaim terkait kualitas batu mulia wajib dapat diverifikasi langsung melalui nomor laporan grading resmi (`certificate_number`). Dilarang mengarang spesifikasi gemologi tanpa data resmi.

---

## 9. Larangan Desain Mutlak (Hard Design Bans)

- ❌ **Dilarang Warna Hitam Pekat `#000000`:** Selalu gunakan `--ql-ink` (`#1A1A1A`).
- ❌ **Dilarang Teks Paragraf Berwarna Emas:** Warna emas hanya untuk aksen terukur dan teks display besar ≥ 24px (atau gunakan `--ql-gold-deep`).
- ❌ **Dilarang Menampilkan Countdown Timer:** Tidak ada penghitung mundur waktu diskon (*no FOMO tactics*).
- ❌ **Dilarang Pop-Up Diskon Intrusif:** Dilarang memunculkan jendela sembulan penawaran diskon email saat pengunjung pertama kali membuka situs.
- ❌ **Dilarang Foto Perhiasan Stok Generik:** Seluruh citra harus merepresentasikan koleksi nyata Bifrost Jewellery.
- ❌ **Dilarang Testimoni Palsu:** Hanya testimoni nyata yang terverifikasi yang boleh ditampilkan.

---

## 10. Dokumen Terkait

- [CODING_RULES.md](./CODING_RULES.md) — Aturan teknis implementasi Tailwind CSS v4
- [STACK.md](./STACK.md) — Definisi font dan dependensi UI
- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) — Filosofi brand dan identitas visual
- [DECISIONS/002-quiet-luxury-design-direction.md](./DECISIONS/002-quiet-luxury-design-direction.md) — Keputusan arsitektur visual quiet luxury
