# 🏛️ ADR 005: Penawaran Ganda Berlian Alami & Lab-Grown dengan Transparansi Sertifikat Penuh

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Founder, Gemological Specialist, Creative Director  
> **Keputusan Terkait:** [ADR 002](./002-quiet-luxury-design-direction.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Lanskap industri berlian global telah mengalami transformasi mendasar. Terdapat dua preferensi pasar yang sama-sama kuat:
1. **Kolektor Berlian Alami (*Natural Diamonds*):** Menghargai kelangkaan geologis miliaran tahun pembentukan bumi, nilai warisan turun-temurun (*heirloom value*), dan prestise tradisional.
2. **Generasi Modern & Peduli Keberlanjutan (*Lab-Grown Diamonds*):** Menginginkan batu permata dengan komposisi kimiawi, optik, dan fisik yang identik 100% dengan berlian alami (kristal karbon murni), namun diciptakan di laboratorium modern berteknologi tinggi (HPHT/CVD) tanpa jejak penambangan bumi, serta memungkinkan kepemilikan ukuran karat yang lebih megah pada kisaran anggaran yang sama.

Beberapa peritel perhiasan mencoba menyamarkan atau mencampuradukkan kedua kategori ini. Hal tersebut merupakan pelanggaran etika serius yang merusak kepercayaan konsumen.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk **Menawarkan Kedua Kategori Berlian (Natural & Lab-Grown) dengan Transparansi Sertifikasi Penuh 100%**.

Prinsip implementasi keputusan ini:
1. **Diferensiasi Eksplisit di Tingkat Database:**
   - Kolom `origin` pada tabel `diamonds` wajib bertipe enum tegas: `'natural'` atau `'lab_grown'`. Tidak ada perhiasan tanpa asal-usul yang jelas.
2. **Sertifikasi Laboratorium Independen:**
   - Setiap berlian berukuran signifikan wajib memiliki sertifikat grading resmi dari laboratorium otoritatif internasional: **GIA** (Gemological Institute of America) atau **IGI** (International Gemological Institute).
   - Nomor laporan grading (`certificate_number`) wajib dicantumkan pada halaman produk (PDP) dan dapat diverifikasi langsung melalui tautan digital resmi lab.
3. **Pembedaan Jelas pada Antarmuka Pengguna:**
   - Filter pencarian katalog (`/pieces`) memiliki opsi filter asal (*Origin Filter*) yang transparan dan mudah dipilih.
   - Setiap kartu perhiasan menyematkan badge minimalis yang menyatakan asal batu mulia secara elegan (*Natural Diamond* atau *Lab-Grown Diamond*).
4. **Narasi Editorial yang Seimbang:**
   - Bagian edukasi editorial menyajikan fakta sains objektif tanpa mendiskreditkan salah satu pihak: menghormati keajaiban geologis berlian alami, sekaligus merayakan kemajuan teknologi penciptaan berlian lab-grown yang etis.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Menjual Berlian Alami Saja (*Natural Diamonds Only*)
- **Kelebihan:** Positioning sangat tradisional dan prestisius bagi kalangan konservatif.
- **Alasan Ditolak:** Menutup peluang besar dari pasar generasi muda, pasangan modern yang mencari cincin pertunangan berukuran karat besar (2–3 karat) dengan harga lebih terjangkau, serta konsumen yang memprioritaskan jejak lingkungan.

### Alternatif B: Menjual Berlian Lab-Grown Saja (*Lab-Grown Only*)
- **Kelebihan:** Narasi keberlanjutan murni (*pure eco-brand*).
- **Alasan Ditolak:** Mengabaikan segmen kolektor bernilai tinggi yang menginginkan kelangkaan alami untuk aset warisan keluarga; membatasi positioning rumah perhiasan mewah adibusana.

### Alternatif C: Mencampurkan Tanpa Label Asal yang Jelas
- **Kelebihan:** Tidak ada.
- **Alasan Ditolak:** Merupakan tindakan penipuan konsumen dan melanggar standar perdagangan gemologi internasional.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Pangsa Pasar yang Luas:** Bifrost Jewellery dapat melayani pelanggan dari seluruh spektrum preferensi dan anggaran tanpa mengorbankan integritas.
- **Reputasi Kepercayaan & Integritas Tinggi:** Transparansi nomor sertifikat GIA/IGI membangun reputasi tak terbantahkan (*unassailable trust*).
- **Inklusivitas Kemewahan Modern:** Pelanggan memiliki kebebasan memilih kemewahan yang selaras dengan nilai personal mereka.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Pemisahan Harga yang Signifikan:** Dua perhiasan dengan desain serupa dapat memiliki selisih harga 3–5 kali lipat karena perbedaan asal berlian. Desain antarmuka dan copywriting harus mampu mengedukasi alasan perbedaan nilai ini secara elegan.
- **Inventori Varian Berlian Ganda:** Pengelolaan stok batu mulia di panel admin membutuhkan penginputan data sertifikat yang teliti.

---

## 5. Referensi & Dokumen Terkait

- [DATABASE.md](../DATABASE.md) — Definisi kolom `origin` dan `certificate_number` tabel `diamonds`
- [GLOSSARY.md](../GLOSSARY.md) — Definisi 4Cs, Natural, dan Lab-Grown
- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Kebutuhan fungsional FR-002 & FR-003
