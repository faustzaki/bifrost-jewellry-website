# 🏛️ ADR 008: Bahasa Konten Publik Bahasa Inggris Penuh & Dokumentasi Sistem Bahasa Indonesia

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Founder, Creative Director, Lead Architect  
> **Keputusan Terkait:** [ADR 002](./002-quiet-luxury-design-direction.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Penetapan bahasa merupakan keputusan strategis krusial yang mempengaruhi citra merek (*brand perception*), kompleksitas teknis arsitektur, dan efisiensi komunikasi tim pengembang:
1. **Citra Merek & Industri Perhiasan Mewah:** Bahasa Inggris adalah *lingua franca* universal dalam dunia perhiasan adibusana (*haute joaillerie*) dan gemologi internasional. Seluruh sertifikat grading resmi (GIA dan IGI) diterbitkan dalam Bahasa Inggris. Target audiens Bifrost Jewellery mencakup masyarakat kelas atas domestik yang terbiasa dengan standar internasional, ekspatriat, dan pembeli global. Penggunaan teks ritel lokal konvensional sering kali menurunkan wibawa eksklusif (*brand prestige*).
2. **Komunikasi & Efisiensi Tim Pengembang Lokal:** Di sisi internal, tim pengembang, arsitek sistem, dan pemilik bisnis berbasis di Indonesia. Instruksi rekayasa, aturan bisnis, dan tata kelola sistem dalam folder `.AGENTS/` membutuhkan kejelasan tanpa ambiguitas untuk memastikan kepatuhan teknis dan kemudahan tinjauan lokal.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan pembagian bahasa yang tegas antara antarmuka publik dan dokumentasi internal:

1. **Seluruh Konten Publik Antarmuka Menggunakan Bahasa Inggris (English Only):**
   - Seluruh teks storefront, menu navigasi, judul editorial, deskripsi perhiasan, spesifikasi 4Cs, formulir checkout, template email transaksional Resend, dan tanda terima pesanan ditulis **100% dalam Bahasa Inggris**.
   - Sistem **TIDAK** menerapkan arsitektur multibahasa (*no i18n multi-language routing*), menjaga arsitektur tetap bersih dan ramping.
2. **Seluruh Dokumentasi Internal AI Agent Menggunakan Bahasa Indonesia:**
   - Seluruh berkas dokumentasi di dalam folder `.AGENTS/` (heading, narasi, penjelasan, tabel panduan, dan aturan arsitektur) ditulis dalam **Bahasa Indonesia**.
   - Seluruh istilah teknis, identifier kode, nama berkas, tipe data, enum database/status, skema Zod, dan token CSS tetap dipertahankan dalam format aslinya (Bahasa Inggris).

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Antarmuka Publik Multibahasa (Bilingual ID + EN via next-intl)
- **Kelebihan:** Fleksibilitas bagi pengguna lokal yang memilih Bahasa Indonesia.
- **Alasan Ditolak:** Menambah kompleksitas arsitektur yang sangat besar (routing `/id/` dan `/en/`, tabel database terjemahan ganda, duplikasi aset narasi editorial, overhead pemeliharaan). Dalam praktiknya, konsumen perhiasan berlian mewah di Indonesia fasih berbahasa Inggris dan lebih mengapresiasi teks bernuansa editorial internasional.

### Alternatif B: Antarmuka Publik Bahasa Indonesia Penuh (Indonesian Only)
- **Kelebihan:** Sepenuhnya berbahasa lokal.
- **Alasan Ditolak:** Merusak estetika *quiet luxury editorial*. Istilah perhiasan seperti *"Round Brilliant Cut"*, *"Prong Solitaire Setting"*, dan *"Eye Clean Clarity"* terdengar canggung dan kehilangan keanggunannya jika dipaksakan diterjemahkan ke dalam bahasa Indonesia sehari-hari.

### Alternatif C: Dokumentasi Sistem Menggunakan Bahasa Inggris Penuh
- **Kelebihan:** Keseragaman bahasa antara kode dan dokumentasi.
- **Alasan Ditolak:** Mengurangi kecepatan pemahaman dan ketajaman evaluasi oleh pemangku kepentingan lokal yang meninjau dokumen fondasi.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Persepsi Kemewahan Kelas Dunia:** Antarmuka publik Bifrost Jewellery langsung memancarkan aura rumah perhiasan mewah internasional yang elegan.
- **Arsitektur Ramping Tanpa Overhead i18n:** Routing URL tetap bersih (`/pieces/[slug]`, bukan `/en/pieces/[slug]`), skema database tidak memerlukan tabel translasi ganda.
- **Ketepatan Eksekusi AI Agent:** Dokumentasi internal berbahasa Indonesia memberikan instruksi yang sangat jelas, terstruktur, dan mudah diaudit oleh tim lokal.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Standar Copywriting Inggris yang Ketat:** Penulis konten wajib menjaga standar tata bahasa Inggris bergaya editorial tinggi (*high editorial prose*) dan dilarang menggunakan terjemahan mesin (*machine translation*) yang kaku.
- **Disiplin Pemisahan Bahasa:** Pengembang harus secara konsisten memisahkan antara narasi dokumentasi internal (Bahasa Indonesia) dan string teks publik di kode komponen (Bahasa Inggris).

---

## 5. Referensi & Dokumen Terkait

- [UI_UX.md](../UI_UX.md) — Aturan tone of voice dan gaya penulisan editorial publik
- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Identitas brand dan target pengguna
- [MANUAL.md](../MANUAL.md) — FAQ keputusan bahasa dokumentasi
