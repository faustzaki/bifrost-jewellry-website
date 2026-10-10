# 📜 FEATURE — Legalitas, Kebijakan & FAQ (LEGAL_AND_FAQ)

> **Status:** Planned  
> **Scope:** Public Legal & Customer Support Pages  
> **Primary Goal:** Memberikan transparansi hukum, perlindungan privasi data pelanggan, kepastian garansi pengembalian dan penyesuaian ukuran gratis (*complimentary resizing*), serta jawaban terstruktur atas pertanyaan umum (FAQ) yang diperkaya dengan skema JSON-LD untuk optimasi mesin pencari Google.

---

## 1. Ringkasan (Overview)

Sebagai jenama perhiasan mewah terpercaya, Bifrost Jewellery menjunjung tinggi kepatuhan hukum, etika kerahasiaan data pribadi klien, dan perlindungan konsumen. Fitur ini menyediakan empat halaman publik standalone yang mudah diakses dari navigasi footer:

1. **Halaman FAQ (`/faq`):** Tanya jawab interaktif seputar keaslian berlian (GIA/IGI), perbedaan natural vs lab-grown, metode pembayaran, keamanan pengiriman berasuransi, dan jadwal kunjungan butik.
2. **Halaman Kebijakan Privasi (`/privacy`):** Pernyataan kepatuhan pengolahan data pribadi pelanggan sesuai regulasi perlindungan data yang berlaku (UU PDP No. 27/2022 di Indonesia dan standar internasional terkait).
3. **Halaman Syarat & Ketentuan (`/terms`):** Ketentuan transaksi penjualan, batasan tanggung jawab, hak kekayaan intelektual desain perhiasan, dan hukum yang berlaku.
4. **Halaman Kebijakan Pengembalian & Garansi (`/returns-and-warranty`):** Prosedur pengembalian barang (*returns*), garansi keaslian seumur hidup, dan ketentuan penyesuaian ukuran cincin gratis pertama kali (*complimentary resizing policy*).

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Calon Pembeli:** Saya ingin membaca FAQ untuk memahami apakah berlian yang saya beli dijamin bersertifikat resmi dan bagaimana pengiriman berasuransi bekerja.
- **Sebagai Pembeli Cincin:** Saya ingin mengetahui syarat dan prosedur penyesuaian ukuran cincin gratis jika ukuran cincin yang saya terima kurang pas di jemari.
- **Sebagai Klien yang Menjaga Privasi:** Saya ingin memastikan bahwa data pribadi saya (alamat, nomor telepon, riwayat pembelian bernilai tinggi) tidak diperjualbelikan kepada pihak ketiga.
- **Sebagai Mesin Pencari Google:** Saya ingin membaca skema metadata terstruktur `FAQPage` dari halaman FAQ agar tanya jawab dapat tampil sebagai cuplikan kaya (*rich snippet*) di hasil pencarian.

---

## 3. Aturan Bisnis (Business Rules)

1. **Ketentuan Kebijakan Pengembalian & Resizing:**
   - **Complimentary Resizing:** Berlaku 1 (satu) kali penyesuaian ukuran cincin gratis dalam kurun waktu 30 hari sejak tanggal barang diterima, dengan batas toleransi ±2 ukuran cincin standar.
   - **Retur Barang:** Karena sifatnya yang bernilai tinggi, karya perhiasan hanya dapat dikembalikan jika terdapat cacat manufaktur bawaan atau ketidakcocokan nomor sertifikat laboratorium (klaim maksimal 7 hari kerja sejak barang diterima dengan segel utuh).
   - Karya pesanan khusus (*bespoke commission*) dan karya dengan grafir inisial kustom tidak dapat dikembalikan kecuali terbukti cacat fisik.
2. **Kepatuhan Privasi Data:**
   - Pengumpulan data dibatasi hanya untuk keperluan pemenuhan pesanan, verifikasi pembayaran, dan janji temu butik.
   - Entitas legal pengelola data dicatat secara formal: `<TBD: nama badan hukum PT Bifrost Jewellery>`.
3. **Penyajian Data Riil Legal:**
   - Informasi hukum dan operasional yang belum difinalisasi oleh konsultan hukum ditandai dengan marker:
     - Badan Hukum: `<TBD: nama badan hukum PT>`
     - Domisili Hukum: `<TBD: yurisdiksi pengadilan negeri>`
     - Alamat Surel Khusus Legal: `<TBD: email resmi tim legal>`
4. **Validitas SEO Skema FAQ:**
   - Halaman FAQ wajib memuat script JSON-LD `FAQPage` di mana setiap elemen pertanyaan (`name`) dan jawaban (`acceptedAnswer.text`) mencerminkan teks yang tampil di layar secara persis tanpa manipulasi kata kunci tersembunyi (*no hidden keywords*).

---

## 4. Integrasi Data (DATABASE.md)

- Pada tahap awal, pertanyaan dan jawaban FAQ serta klausul hukum dapat dimuat dari tabel referensi atau file konfigurasi konten terkelola:
  - `faqs`: Entitas basis data untuk FAQ (`id`, `question`, `answer`, `category`, `display_order`, `is_active`).
    - Kategori FAQ: `diamonds_and_gemology`, `ordering_and_payment`, `shipping_and_insurance`, `resizing_and_warranty`, `boutique_visits`.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

Halaman-halaman legal ini diakses melalui rute publik statis (SSG/ISR):
- `GET /faq`
- `GET /privacy`
- `GET /terms`
- `GET /returns-and-warranty`

Daftar FAQ dapat diambil secara langsung dalam React Server Component:
```typescript
const faqs = await prisma.faq.findMany({
  where: { isActive: true },
  orderBy: { displayOrder: 'asc' }
});
```

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `LegalLayout`: Tata letak dua kolom editorial dengan navigasi daftar isi samping (*Sticky Table of Contents / TOC*) di kolom kiri dan pasal-pasal hukum di kolom kanan.
- `Accordion`: Komponen akordeon pertanyaan FAQ dengan animasi ekspansi dan kontraksi halus berbasis CSS/Framer Motion, ikon plus/minus minimalis bergaris rambut (`--ql-line`), dan kontras teks yang memenuhi standar AA.
- `FaqCategoryTabs`: Tab filter kategori pertanyaan yang tenang di bagian atas halaman FAQ.
- `FaqJsonLd`: Komponen injeksi skrip JSON-LD `schema.org/FAQPage` ke tag `<head>`.
- `ContactConciergeCTA`: Banner penutup halaman yang mengajak pengguna menghubungi tim concierge jika pertanyaan spesifik mereka belum terjawab.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Seluruh rute legal (`/faq`, `/privacy`, `/terms`, `/returns-and-warranty`) dapat diakses secara publik dan mengembalikan respon 200 OK.
- [ ] Akordeon FAQ membuka dan menutup pertanyaan dengan mulus tanpa pergeseran layout yang mendadak, serta dapat dioperasikan penuh melalui navigasi keyboard (Enter / Space bar untuk toggle, tab order yang benar).
- [ ] Navigasi TOC pada halaman Syarat & Ketentuan dan Privasi bergulir secara halus ke section terkait dengan offset jarak navbar yang tepat (`scroll-margin-top`).
- [ ] Skrip JSON-LD pada halaman `/faq` lolos validasi tanpa kesalahan pada Google Rich Results Test dan schema.org validator.
- [ ] Ketentuan penyesuaian ukuran cincin gratis (*complimentary resizing*) dan syarat pengembalian perhiasan diuraikan secara lugas tanpa ambiguitas hukum.

---

## 8. Dokumen Terkait

- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Tautan jaminan garansi dan pengembalian
- [CART_CHECKOUT.md](./CART_CHECKOUT.md) — Persetujuan syarat & ketentuan saat proses checkout
- [UI_UX.md](../UI_UX.md) — Standar aksesibilitas akordeon dan warna teks
