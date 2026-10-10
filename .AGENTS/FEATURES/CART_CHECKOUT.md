# 💎 FEATURE — Keranjang Belanja & Checkout (CART_CHECKOUT)

> **Status:** Planned  
> **Scope:** Public + Customer Commerce Flow  
> **Primary Goal:** Menyediakan alur belanja tanpa hambatan melalui keranjang belanja persisten, dukungan pembelian tanpa akun (*guest checkout*) maupun akun terdaftar, perhitungan ongkos kirim dan asuransi, revalidasi integritas harga secara ketat di sisi server, serta penerbitan token transaksi Midtrans Snap.

---

## 1. Ringkasan (Overview)

Fitur Keranjang & Checkout dirancang untuk memberikan kenyamanan belanja perhiasan mewah dengan transparansi biaya total sejak awal. Pengguna dapat meninjau item yang dipilih melalui *Cart Drawer* geser yang elegan atau halaman ringkasan keranjang penuh. 

Sistem mendukung penuh **Guest Checkout**: pelanggan tidak diwajibkan mendaftar akun di muka untuk melakukan transaksi, namun diberikan opsi pendaftaran instan setelah pembayaran selesai untuk kemudahan pelacakan pesanan. Seluruh perhitungan subtotal, konversi mata uang (IDR / USD), asuransi pengiriman bernilai tinggi (*high-value courier insurance*), dan pembuatan entitas pesanan baru diverifikasi dan dikalkulasi secara mutlak di server guna mencegah manipulasi sisi klien.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pengunjung:** Saya ingin menambahkan karya perhiasan ke dalam keranjang belanja dan melihat ringkasan item (nama karya, ukuran, logam, dan harga) melalui *Cart Drawer* tanpa harus meninggalkan halaman yang sedang saya lihat.
- **Sebagai Pembeli Pertama Kali:** Saya ingin melakukan checkout secara langsung hanya dengan memasukkan alamat email dan nomor telepon tanpa dipaksa membuat akun atau mengingat kata sandi.
- **Sebagai Pelanggan Terdaftar:** Saya ingin menggunakan buku alamat yang telah tersimpan di akun saya agar proses pengisian formulir pengiriman berlangsung lebih cepat.
- **Sebagai Pembeli Perhiasan Mewah:** Saya ingin melihat rincian biaya pengiriman berasuransi penuh secara terpisah dan jelas sebelum menekan tombol pembayaran akhir.
- **Sebagai Pembeli:** Saya ingin memilih opsi antara pembayaran digital terintegrasi (Midtrans Snap) atau transfer rekening bank manual dengan instruksi yang terperinci.

---

## 3. Aturan Bisnis (Business Rules)

1. **Revalidasi Harga Sisi Server (Server-Side Price Invariance):**
   - Klien hanya mengirimkan pasangan `{ variantId, quantity }`.
   - Server wajib mengambil harga resmi terkini dari basis data (`pieces.price_idr_minor` atau `pieces.price_usd_minor`).
   - Nilai harga yang dikirim dari klien (jika ada) sama sekali tidak dipercaya dan diabaikan.
2. **Ketersediaan dan Batas Kuantitas:**
   - Karena perhiasan bernilai tinggi dan sebagian besar dibuat terbatas atau *one-of-a-kind*, batas kuantitas per baris pesanan default adalah 1 unit kecuali dikonfigurasi lain.
   - Sebelum membuat order, server memvalidasi bahwa stok varian (`piece_variants.stock >= quantity`) mencukupi.
3. **Penetapan Mata Uang & Biaya:**
   - Mata uang pesanan (`currency`) dikunci pada saat pembuatan order (`IDR` atau `USD`).
   - Biaya pengiriman dan asuransi perhiasan wajib dihitung berdasarkan tujuan pengiriman:
     - Domestik Indonesia: Kurir khusus berasuransi perhiasan (*valuable goods*), biaya dasar dihitung otomatis.
     - Internasional: Kurir kargo aman internasional (misal: FedEx International Priority dengan asuransi pihak ketiga).
4. **Siklus Pembentukan Pesanan (Order Initiation):**
   - Transaksi checkout yang valid akan membuat entitas `orders` dengan status awal `pending_payment`.
   - Kode unik pesanan (`order_number`) digenerasi dengan format acak dan aman: `BJ-YYYYMMDD-XXXX` (misal: `BJ-20261011-8F3A`).
   - Sistem menerbitkan sesi pembayaran ke Midtrans Snap dan mengembalikan `snap_token` serta URL redirect.
   - Masa berlaku pembayaran (*payment expiry window*) dikonfigurasi selama 24 jam untuk transfer bank / VA, dan 1 jam untuk kartu kredit/QRIS.

---

## 4. Integrasi Data (DATABASE.md)

Fitur ini mengelola dan memperbarui data pada tabel-tabel berikut:

- `carts`: Entitas keranjang belanja sesi tamu atau user terdaftar (`id`, `user_id`, `session_token`, `currency`).
- `cart_lines`: Rincian item dalam keranjang (`id`, `cart_id`, `piece_id`, `variant_id`, `quantity`).
- `orders`: Entitas pesanan utama (`id`, `order_number`, `user_id`, `guest_email`, `status`, `currency`, `subtotal_minor`, `shipping_minor`, `total_minor`, `shipping_address_id`, `notes`).
- `order_lines`: *Snapshot* permanen item pesanan beserta harga saat transaksi dibuat (`id`, `order_id`, `piece_id`, `variant_id`, `quantity`, `unit_price_minor_idr`, `unit_price_minor_usd`).
- `addresses`: Alamat pengiriman fisik penerima (`id`, `user_id`, `recipient_name`, `phone`, `street_line1`, `street_line2`, `city`, `province_or_state`, `postal_code`, `country_code`).
- `payments`: Rekaman awal pembayaran dengan status `pending`.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Checkout API Endpoint

- **`POST /api/checkout`**
  - **Headers:** `Content-Type: application/json`
  - **Request Body (Zod Validated):**
    ```json
    {
      "cartLines": [
        {
          "variantId": "var-06",
          "quantity": 1
        }
      ],
      "currency": "IDR",
      "customer": {
        "email": "customer@example.com",
        "fullName": "Jane Doe",
        "phone": "+628123456789"
      },
      "shippingAddress": {
        "recipientName": "Jane Doe",
        "phone": "+628123456789",
        "streetLine1": "Jl. Senopati No. 45",
        "streetLine2": "Apt 12B",
        "city": "Jakarta Selatan",
        "provinceOrState": "DKI Jakarta",
        "postalCode": "12190",
        "countryCode": "ID"
      },
      "paymentMethod": "midtrans_snap",
      "notes": "Harap sertakan amplop kado eksklusif Bifrost"
    }
    ```
  - **Status 201 Created Response:**
    ```json
    {
      "success": true,
      "data": {
        "orderNumber": "BJ-20261011-8F3A",
        "orderId": "ord-7f8a-9b0c-1d2e",
        "totalMinor": 18550000000,
        "currency": "IDR",
        "snapToken": "e72f9b10-6c3d-4e5a-8b9c-0d1e2f3a4b5c",
        "redirectUrl": "https://app.sandbox.midtrans.com/snap/v2/vtweb/e72f9b10-6c3d-4e5a-8b9c-0d1e2f3a4b5c"
      }
    }
    ```
  - **Status 409 Conflict (Stok Habis / Perubahan Status):**
    ```json
    {
      "success": false,
      "error": {
        "code": "OUT_OF_STOCK",
        "message": "Maaf, varian perhiasan yang dipilih saat ini tidak lagi tersedia dalam jumlah yang diminta."
      }
    }
    ```

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `CartDrawer`: Panel laci keranjang samping dengan backdrop redup, menampilkan daftar item dengan thumbnail rapi, tombol hapus senyap, subtotal real-time, dan tombol checkout "Proceed to Checkout".
- `CheckoutStepper`: Indikator langkah tenang tanpa visual mencolok (Langkah 1: Pengiriman → Langkah 2: Pembayaran → Langkah 3: Konfirmasi).
- `ShippingAddressForm`: Formulir masukan alamat pengiriman dengan validasi input instan dan auto-complete kode pos.
- `OrderSummaryCard`: Panel sticky di sisi kanan yang memperlihatkan rincian item, subtotal, biaya pengiriman aman berasuransi, dan grand total.
- `PaymentMethodSelector`: Pilihan radio kustom antara "Pembayaran Online Instan (Kartu Kredit / VA / QRIS via Midtrans)" dan "Transfer Bank Manual (BCA / Mandiri)".
- `PostCheckoutRegisterModal`: Modal yang muncul di halaman sukses pembayaran yang mengajak pembeli tamu membuat kata sandi untuk mengonversi data menjadi akun terdaftar dengan satu klik.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Pengguna dapat menambah, mengubah kuantitas, dan menghapus item dari keranjang dengan pembaruan subtotal otomatis tanpa lag.
- [ ] Keranjang tamu tersimpan dalam cookie sesi terenkripsi sehingga tidak hilang saat pengguna me-refresh halaman atau berpindah tautan.
- [ ] Panggilan `POST /api/checkout` memvalidasi ulang harga per item ke basis data dan menolak permintaan jika stok habis dengan kode status 409.
- [ ] Form checkout tamu berhasil memproses pesanan tanpa meminta kata sandi akun.
- [ ] Pesanan yang berhasil dibuat tersimpan di basis data dengan status `pending_payment` dan relasi `order_lines` yang merekam harga snapshot.
- [ ] Untuk metode Midtrans Snap, sistem berhasil menerima `snapToken` yang valid dari API gateway dan memicu popup pembayaran Snap di peramban klien.

---

## 8. Dokumen Terkait

- [PAYMENT.md](./PAYMENT.md) — Alur integrasi Midtrans Snap dan transfer manual
- [ORDERS.md](./ORDERS.md) — Pelacakan status dan siklus hidup pesanan
- [SERVER_ACTION_PATTERN.md](../PATTERNS/SERVER_ACTION_PATTERN.md) — Pola mutasi data checkout aman
