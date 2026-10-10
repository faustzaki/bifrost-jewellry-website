# 🏛️ ADR 004: Dukungan Penuh Guest Checkout dengan Pembuatan Akun Opsional

> **Status:** Accepted  
> **Tanggal:** 2026-10-11  
> **Pengambil Keputusan:** Lead Architect, Head of Operations  
> **Keputusan Terkait:** [ADR 001](./001-nextjs15-app-router-postgres.md), [ADR 003](./003-hybrid-commerce-model.md)

---

## 1. Konteks & Masalah (Context & Problem Statement)

Kewajiban mendaftarkan akun dan mengingat kata sandi sebelum checkout merupakan penyebab utama pembatalan transaksi (*cart abandonment*) pada e-commerce modern. Dalam industri barang mewah, pembeli sering kali memprioritaskan privasi, kecepatan transaksi, dan kenyamanan tanpa hambatan (*frictionless experience*). Memaksa pembeli mengisi formulir pendaftaran akun berlapis sebelum dapat membayar dapat menurunkan tingkat konversi secara signifikan.

Di sisi lain, akun terdaftar tetap memberikan nilai tambah nyata bagi pelanggan setia: riwayat pembelian perhiasan, nomor resi pengiriman, daftar keinginan (*wishlist*), dan penyimpanan buku alamat pengiriman.

---

## 2. Keputusan yang Diambil (Decision)

Kami memutuskan untuk mendukung **Guest Checkout Penuh dengan Pembuatan Akun Opsional**.

Rincian implementasi keputusan ini:
1. **Guest Checkout Tanpa Hambatan:**
   - Pengunjung dapat menyelesaikan proses checkout hanya dengan memasukkan nama, alamat email aktif, nomor telepon/WhatsApp, dan alamat pengiriman.
   - Kolom `user_id` pada tabel `orders` bersifat opsional (`NULLABLE`). Data tamu disimpan pada kolom `guest_email`, `guest_name`, dan `guest_phone`.
2. **Pelacakan Pesanan Tamu (*Guest Order Tracking*):**
   - Tamu dapat melacak status pesanan secara mandiri di halaman `/orders/track` menggunakan kombinasi nomor pesanan resmi (`order_number`) dan alamat email.
3. **Pendaftaran Akun Opsional Pasca-Pembelian (*Post-Purchase Account Creation*):**
   - Pada halaman konfirmasi pembayaran sukses (`/checkout/success`) dan dalam email tanda terima pesanan, tamu diberikan opsi satu klik untuk membuat kata sandi dan mengaktifkan akun resmi.
   - Jika tamu membuat akun di kemudian hari menggunakan email yang sama, sistem secara otomatis mengaitkan riwayat pesanan tamu sebelumnya ke akun baru tersebut.
4. **Akun Terdaftar:** Pengguna yang memilih login tetap mendapatkan kemudahan pengisian form alamat otomatis, sinkronisasi wishlist antar perangkat, dan riwayat pesanan terpusat.

---

## 3. Alternatif yang Dipertimbangkan (Alternatives Considered)

### Alternatif A: Pendaftaran Akun Wajib Sebelum Checkout (*Mandatory Registration*)
- **Kelebihan:** Seluruh pesanan terikat rapi pada entitas `users`, manajemen relasi pelanggan (CRM) terpusat sejak awal.
- **Alasan Ditolak:** Meningkatkan angka *drop-off* checkout hingga 25–30%. Pembeli kado dadakan atau pembeli perhiasan pertama kali akan merasa terbebani oleh proses verifikasi email/kata sandi.

### Alternatif B: Tamu Murni Tanpa Fitur Akun Sama Sekali (*Guest-Only Platform*)
- **Kelebihan:** Sangat sederhana, tidak memerlukan modul otentikasi pelanggan atau manajemen sesi akun.
- **Alasan Ditolak:** Menyulitkan pelanggan loyal (*repeat collectors*) yang harus mengetik ulang alamat pengiriman panjang setiap kali memesan; fitur *wishlist* tidak dapat disinkronkan antar perangkat.

---

## 4. Konsekuensi (Consequences)

### Konsekuensi Positif (Benefits)
- **Rasio Penyelesaian Checkout Maksimal:** Alur pembelian sangat singkat dan mulus, memaksimalkan konversi belanja.
- **Menghormati Privasi Pelanggan:** Pembeli tidak merasa dipaksa menyerahkan data kredensial akun jika hanya ingin bertransaksi satu kali.
- **Fleksibilitas Data:** Sistem tetap mampu mengakomodasi kebutuhan pembeli tamu maupun pengguna terdaftar secara harmonis.

### Konsekuensi Negatif / Kompromi (Trade-offs)
- **Kompleksitas Skema Pesanan:** Tabel `orders` harus menangani dua skenario (pesanan milik `user_id` atau pesanan milik data `guest_*`).
- **Logika Agregasi Akun:** Diperlukan logika latar belakang untuk menghubungkan pesanan tamu lama ketika pemilik email tersebut memutuskan mendaftarkan akun di kemudian hari.

---

## 5. Referensi & Dokumen Terkait

- [DATABASE.md](../DATABASE.md) — Skema kolom `user_id` nullable pada tabel `orders`
- [API_CONTRACT.md](../API_CONTRACT.md) — Payload endpoint `POST /api/checkout`
- [PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) — Kebutuhan fungsional FR-004 & FR-008
