# 🏛️ ADR 002: Arah Desain Visual Editorial Quiet Luxury & Aturan Tipografi

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Creative Director, Lead Architect, UI/UX Designer  
> **Keputusan Terkait:** [ADR 001](./001-nextjs15-app-router-postgres.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Industri perhiasan berlian mewah menuntut tingkat kepercayaan dan wibawa estetika yang luar biasa tinggi. Mayoritas situs e-commerce retail konvensional menggunakan elemen visual agresif: banner diskon berwarna merah mencolok, penghitung waktu mundur (*countdown timer*), pop-up penawaran instan, dan label promo berkedip. Dalam konteks perhiasan adibusana (*haute joaillerie*), taktik pemasaran ritel berteriak tersebut merusak persepsi kemewahan (*brand equity*) dan membuat perhiasan bernilai puluhan juta rupiah terlihat seperti komoditas murah.

Bifrost Jewellery membutuhkan identitas visual yang memancarkan ketenangan, integritas, dan keabadian nilai (*timeless elegance*), yang selaras dengan ekspektasi kolektor dan pembeli perhiasan kelas atas.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk menetapkan arah desain visual **Quiet Luxury Editorial**.

Prinsip dan aturan visual yang dikunci dalam sistem:
1. **Palet Warna Hangat Berkontras Tinggi (WCAG AA):**
   - Latar belakang kanvas hangat off-white `--ql-canvas` (`#FAF8F5`) berpadu dengan permukaan bersih `--ql-surface` (`#FFFFFF`).
   - Teks gelap pekat `--ql-ink` (`#1A1A1A`) memberikan kontras luar biasa 16.42:1 (lulus WCAG AAA). Dilarang memakai warna hitam murni `#000000`.
   - Warna emas `--ql-gold` (`#9C7C3F`) dikhususkan untuk ornamen dan teks display besar (≥ 24px). Untuk teks aksen emas ukuran kecil/normal, wajib menggunakan `--ql-gold-deep` (`#7C6236`) agar lolos standar kontras WCAG AA (5.42:1).
2. **Kombinasi Tipografi Editorial:**
   - Display serif: `Cormorant Garamond` (bobot 400 & 500) untuk tajuk utama editorial dan nama perhiasan.
   - Body sans-serif: `Inter` untuk kejelasan teks antarmuka, deskripsi, dan formulir.
   - Monospace: `JetBrains Mono` untuk nomor sertifikat GIA/IGI, nomor SKU, dan kode pesanan.
3. **Komposisi Tata Letak & Spacing:**
   - Whitespace luas (*py-24* hingga *py-32* di desktop) sebagai elemen pembatas alami.
   - Sudut kartu berkarakter arsitektural tajam `rounded-[2px]` dengan garis pembatas 1px (*hairline border* `--ql-line`).
   - Tombol aksi berbentuk kapsul halus (*pill button*).
4. **Larangan Desain Mutlak:** Dilarang menggunakan pop-up diskon, countdown timer, gradien emas norak, dan tanda seru pada copywriting.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Bold Retail E-Commerce (Gaya Retail Marketplace Konvensional)
- **Kelebihan:** Memicu konversi cepat melalui psikologi urgensi (*FOMO*, banner diskon besar, flash sale).
- **Alasan Ditolak:** Menghancurkan positioning eksklusif perhiasan mewah. Pelanggan berlian bernilai tinggi mencari rasa aman dan prestise, bukan diskon murah.

### Alternatif B: Dark Glamour / Black Velvet (Serba Hitam Pekat dengan Emas Berkilau)
- **Kelebihan:** Kesan misterius dan dramatis.
- **Alasan Ditolak:** Menyebabkan kelelahan mata (*visual fatigue*), keterbacaan teks panjang buruk, kontras warna sulit distandarisasi untuk aksesibilitas WCAG, dan terkesan klise/kuno.

### Alternatif C: Cold Minimalist SaaS (Gaya Antarmuka SaaS Abu-abu Dingin)
- **Kelebihan:** Sangat bersih dan fungsional.
- **Alasan Ditolak:** Terlalu kaku dan steril (*lacks warmth and emotion*). Perhiasan adalah produk emosional yang membutuhkan kehangatan visual (*warm editorial soul*).

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Diferensiasi Pasar yang Kuat:** Bifrost Jewellery langsung tampil menonjol sebagai butik perhiasan berwibawa tinggi, berbeda dari toko perhiasan retail biasa.
- **Aksesibilitas Terjamin (WCAG AA):** Seluruh token warna telah diverifikasi matematis sehingga ramah bagi seluruh pengguna.
- **Fokus Tajam pada Produk:** Fotografi perhiasan menjadi pusat perhatian tanpa gangguan grafis yang bising.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Ketergantungan Tinggi pada Kualitas Aset Foto:** Desain quiet luxury menuntut fotografi perhiasan asli beresolusi tinggi dengan pencahayaan profesional; aset foto amatir akan terlihat sangat kontras.
- **Disiplin Ketat pada Copywriting:** Tim konten dilarang keras menggunakan teknik copywriting ritel persuasif murahan.

---

## 5. Referensi & Dokumen Terkait

- [UI_UX.md](../UI_UX.md) — Panduan desain quiet luxury dan tabel token warna
- [CODING_RULES.md](../CODING_RULES.md) — Penegakan token semantik di Tailwind v4
- [STACK.md](../STACK.md) — Konfigurasi font Cormorant Garamond, Inter, & JetBrains Mono
