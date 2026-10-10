# 👤 FEATURE — Akun Pelanggan & Wishlist (ACCOUNT)

> **Status:** Planned  
> **Scope:** Customer Account & Personalization (Authenticated Customer)  
> **Primary Goal:** Memberikan ruang pribadi bagi pelanggan terdaftar untuk mengelola profil, menyimpan buku alamat pengiriman, meninjau riwayat pesanan, serta menyusun daftar mahakarya impian (*wishlist*).

---

## 1. Ringkasan (Overview)

Sesuai filosofi Bifrost Jewellery, kepemilikan akun bersifat sepenuhnya **opsional** dan tidak pernah dipaksakan saat checkout. Namun, bagi klien yang memilih untuk membuat akun, modul Akun Pelanggan menghadirkan pengalaman personalisasi tingkat tinggi yang tenang dan fungsional.

Melalui portal `/account`, pelanggan dapat memperbarui rincian kontak pribadi, mengatur alamat pengiriman utama dan sekunder untuk mempermudah checkout di masa depan, meninjau seluruh riwayat faktur pesanan berhiaskan nomor sertifikat berlian, serta mengelola koleksi *Wishlist* karya perhiasan yang diminati.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pelanggan Terdaftar:** Saya ingin melihat profil pribadi saya dan memperbarui nomor telepon atau nama lengkap saya dengan aman.
- **Sebagai Pembeli Rutin:** Saya ingin menyimpan beberapa alamat pengiriman (misal: Rumah, Kantor, atau Alamat Hadiah) dan memilih salah satu sebagai alamat utama, agar saya tidak perlu mengetik ulang saat berbelanja.
- **Sebagai Penikmat Koleksi:** Saya ingin menyimpan karya perhiasan tertentu ke dalam *Wishlist* saya dengan satu klik ikon hati/bintang, dan melihat kembali karya-karya tersebut kapan saja.
- **Sebagai Kolektor:** Saya ingin melihat daftar lengkap seluruh pesanan perhiasan yang pernah saya beli beserta salinan digital sertifikat laboratoriumnya.

---

## 3. Aturan Bisnis (Business Rules)

1. **Prinsip Akun Opsional (Optional Account Model):**
   - Pengguna dapat mendaftar kapan saja melalui `/register` atau secara otomatis membuat akun di akhir alur pembayaran checkout tamu dengan menentukan kata sandi.
   - Pesanan tamu yang menggunakan email yang sama dapat ditautkan (*linked*) ke akun baru setelah email pelanggan terverifikasi.
2. **Buku Alamat (Address Book Constraints):**
   - Seorang pengguna dapat menyimpan maksimal 5 alamat pengiriman.
   - Tepat satu alamat harus ditandai sebagai alamat utama (`is_default = true`).
   - Jika pengguna menghapus alamat utama dan masih memiliki alamat lain, sistem secara otomatis menetapkan alamat tertua berikutnya sebagai alamat utama baru.
3. **Wishlist Dinamis:**
   - Karya perhiasan yang dihapus dari katalog atau diarsipkan oleh admin tetap ditandai sebagai "Archived / Tidak Tersedia" pada tampilan wishlist pengguna.
   - Penambahan ke wishlist dari pengguna yang belum login (*guest*) disimpan sementara di `localStorage` peramban, dan disinkronkan (*merged*) ke basis data saat pengguna berhasil login.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini berinteraksi dengan:

- `users`: Informasi akun utama (`id`, `email`, `full_name`, `phone`, `role`, `created_at`).
- `addresses`: Buku alamat pengiriman (`id`, `user_id`, `recipient_name`, `phone`, `street_line1`, `street_line2`, `city`, `province_or_state`, `postal_code`, `country_code`, `is_default`).
- `orders` & `order_lines`: Riwayat transaksi dan faktur yang terkait dengan `user_id`.
- `wishlist_items`: Relasi item yang disimpan pelanggan (`id`, `user_id`, `piece_id`, `created_at`).
- `pieces` & `piece_images`: Data katalog untuk merender kartu perhiasan di wishlist.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Server Actions Portal Akun

- **`updateProfile({ fullName: string, phone?: string })`**
  - Memperbarui data pengguna pada sesi aktif.
- **`saveAddress({ id?: string, recipientName: string, phone: string, streetLine1: string, streetLine2?: string, city: string, provinceOrState: string, postalCode: string, countryCode: string, isDefault: boolean })`**
  - Menambah atau memperbarui rekaman alamat di tabel `addresses`.
- **`deleteAddress(addressId: string)`**
  - Menghapus alamat milik pengguna yang sedang login.
- **`toggleWishlist(pieceId: string)`**
  - Menambah atau menghapus karya perhiasan dari tabel `wishlist_items`.
- **`syncGuestWishlist(pieceIds: string[])`**
  - Menggabungkan daftar ID perhiasan dari penyimpanan lokal peramban ke akun pengguna saat login perdana.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `AccountSidebar`: Menu navigasi portal pelanggan yang minimalis (Profil, Riwayat Pesanan, Buku Alamat, Wishlist, Keluar).
- `AddressCard`: Kartu beralas `--ql-surface` dengan border rambut halus `--ql-line`, menampilkan nama penerima, alamat berformat rapi, badge "Alamat Utama", dan tombol aksi Edit / Hapus.
- `WishlistGrid`: Tata letak kartu perhiasan yang bersih dengan tombol cepat "Move to Bag" atau "Remove".
- `ProfileEditForm`: Formulir pembaruan profil yang elegan dengan validasi real-time.
- `OrderHistoryTable`: Daftar pesanan terperinci dengan nomor pesanan terformat JetBrains Mono, tanggal pembelian, nominal total, status pesanan, dan tombol "Lihat Detail".

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Pengguna yang belum terotentikasi otomatis dialihkan ke halaman login saat mencoba mengakses rute `/account/*`.
- [ ] Pengguna dapat memperbarui nama lengkap dan nomor telepon, dan data langsung ter-revalidasi di sesi peramban.
- [ ] Menambahkan alamat baru dengan opsi `is_default = true` secara otomatis menonaktifkan status default pada alamat pengguna lainnya.
- [ ] Mengklik tombol wishlist pada kartu perhiasan langsung memperbarui status ikon dan menyinkronkan data ke basis data tanpa reload halaman.
- [ ] Item wishlist yang disimpan oleh tamu berhasil disinkronkan ke akun setelah login dilakukan.
- [ ] Riwayat pesanan hanya menampilkan pesanan yang memiliki `user_id` milik pengguna yang sedang login.

---

## 8. Dokumen Terkait

- [AUTH.md](./AUTH.md) — Otentikasi dan sesi Auth.js v5
- [ORDERS.md](./ORDERS.md) — Detail status pesanan pelanggan
- [SERVER_ACTION_PATTERN.md](../PATTERNS/SERVER_ACTION_PATTERN.md) — Pola Server Action dengan validasi sesi Zod
