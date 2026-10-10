# 🏛️ ADR 006: Integrasi Pembayaran Midtrans Snap & Opsi Transfer Bank Manual

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Lead Architect, Head of Finance & Operations  
> **Keputusan Terkait:** [ADR 001](./001-nextjs15-app-router-postgres.md), [ADR 003](./003-hybrid-commerce-model.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Pasar e-commerce perhiasan mewah di Indonesia memiliki karakteristik pembayaran yang unik:
1. **Transaksi Menengah (< Rp 30.000.000):** Pembeli menginginkan metode pembayaran digital instan yang terpercaya: Virtual Account perbankan utama (BCA, Mandiri, BNI, BRI), QRIS, maupun Kartu Kredit dengan verifikasi keamanan 3D Secure.
2. **Transaksi Nilai Sangat Tinggi (*High-Ticket Transactions* > Rp 50.000.000):** Sebagian besar nasabah perbankan prioritas memiliki limit pembayaran kartu kredit atau limit transaksi harian payment gateway. Untuk perhiasan bernilai tinggi, mereka lebih memilih melakukan transfer bank langsung antar-rekening koran resmi perusahaan melalui internet banking korporat (*corporate bank transfer*).
3. **Kepatuhan Regulasi & Standar PCI-DSS:** Menyimpan atau memproses nomor kartu kredit secara langsung di server aplikasi memerlukan sertifikasi audit PCI-DSS yang sangat mahal dan rumit.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk mengintegrasikan **Midtrans Snap sebagai Gateway Pembayaran Digital** serta menyediakan **Jalur Transfer Bank Manual Terverifikasi**.

Arsitektur pembayaran diatur sebagai berikut:
1. **Midtrans Snap untuk Pembayaran Otomatis:**
   - Aplikasi menggunakan pustaka `midtrans-client` di sisi server untuk meminta token transaksi (`snap_token`).
   - Pelanggan menyelesaikan pembayaran melalui jendela modal hosted aman milik Midtrans Snap (mendukung VA BCA/Mandiri/BNI/BRI, QRIS, dan Kartu Kredit).
   - Server aplikasi tidak pernah menyentuh data kartu kredit pelanggan, sehingga kepatuhan PCI-DSS sepenuhnya dialihdayakan ke Midtrans.
   - Endpoint webhook `POST /api/payments/midtrans/webhook` memvalidasi tanda tangan kriptografi SHA-512 dan menerapkan prinsip idempotensi untuk mencegah proses ganda.
2. **Jalur Transfer Bank Manual (*Manual Bank Transfer*):**
   - Opsi pembayaran resmi bagi pelanggan yang ingin mentransfer dana langsung ke rekening bank resmi butik (`<TBD: rekening bank transfer manual>`).
   - Pelanggan mengunggah bukti transfer melalui antarmuka web, dan status pembayaran diset ke `awaiting_verification`.
   - Tim finance memverifikasi mutasi rekening nyata di internet banking, lalu menyetujui transaksi melalui tombol verifikasi di panel admin internal.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Stripe Payment Gateway
- **Kelebihan:** Standar emas gateway global, dokumentasi developer terbaik di dunia.
- **Alasan Ditolak:** Dukungan metode pembayaran lokal Indonesia terbatas. Stripe tidak memiliki integrasi langsung dengan Virtual Account BCA domestik atau QRIS lokal yang merupakan metode pembayaran dominan bagi pembeli di Indonesia.

### Alternatif B: Xendit Payment Gateway
- **Kelebihan:** Dukungan pembayaran lokal Indonesia yang sangat kuat.
- **Alasan Ditolak:** Midtrans Snap dinilai memiliki modal antarmuka pembayaran pop-up (*in-app embedded modal*) yang lebih mulus dan telah memiliki rekam jejak integrasi teruji pada proyek-proyek ekosistem tim.

### Alternatif C: Hanya Menyediakan Transfer Bank Manual Tanpa Gateway
- **Kelebihan:** Bebas biaya administrasi potongan payment gateway.
- **Alasan Ditolak:** Menyebabkan beban operasional verifikasi manual yang lambat untuk setiap pembelian perhiasan kecil/menengah; menurunkan konversi bagi pembeli yang ingin verifikasi instan tengah malam.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Cakupan Pembayaran Menyeluruh:** Melayani pembeli instan (via Midtrans Snap) dan pembeli perhiasan eksklusif bernilai tinggi (via transfer manual korporat).
- **Keamanan Finansial Terjamin:** PCI-DSS scope berada di luar server aplikasi.
- **Ketahanan Terhadap Manipulasi:** Verifikasi SHA-512 signature key mencegah manipulasi notifikasi status pembayaran oleh pihak ketiga jahat.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Biaya Layanan Gateway:** Transaksi via Midtrans dikenakan biaya pemrosesan standar gateway per transaksi.
- **Kebutuhan Verifikasi Manusia:** Alur transfer bank manual membutuhkan ketelitian tim admin finance untuk mencocokkan mutasi bank sebelum mengubah status pesanan menjadi `paid`.

---

## 5. Referensi & Dokumen Terkait

- [API_CONTRACT.md](../API_CONTRACT.md) — Kontrak webhook Midtrans & endpoint transfer manual
- [ARCHITECTURE.md](../ARCHITECTURE.md) — Sequence diagram alur checkout dan webhook
- [DATABASE.md](../DATABASE.md) — Tabel `orders` dan `payments`
