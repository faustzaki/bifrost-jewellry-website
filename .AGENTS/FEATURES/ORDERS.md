# 📦 FEATURE — Manajemen & Pelacakan Pesanan (ORDERS)

> **Status:** Planned  
> **Scope:** Order Lifecycle & Tracking (Public Guest Lookup + Customer Account + Admin Fulfilment)  
> **Primary Goal:** Mengelola seluruh tahapan siklus hidup pesanan perhiasan dari pemesanan awal hingga pengiriman selesai, serta menyediakan fasilitas pelacakan status pesanan yang aman bagi pelanggan tamu (*guest lookup*) dan pelanggan terdaftar.

---

## 1. Ringkasan (Overview)

Pesanan perhiasan bernilai tinggi memerlukan komunikasi transparan dan pembaruan berkala. Modul Pesanan Bifrost Jewellery memastikan setiap tahap pengerjaan mahakarya perhiasan—mulai dari verifikasi pembayaran, penyesuaian ukuran (*sizing*), pengemasan khusus anti-rusak, hingga pengawalan pengiriman kurir berasuransi—tercatat secara sistematis.

Pelanggan yang berbelanja tanpa akun (*guest*) dapat memeriksa status pesanan kapan saja melalui halaman publik `/orders/track` hanya dengan memasukkan Nomor Pesanan (`order_number`) dan Alamat Email yang digunakan saat checkout. Bagi pengguna yang memiliki akun, riwayat seluruh pesanan dapat diakses langsung pada dashboard akun.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pembeli Tamu:** Saya ingin melacak status pengerjaan dan pengiriman pesanan saya menggunakan nomor pesanan dan email saya tanpa perlu mendaftar akun atau mengingat kata sandi.
- **Sebagai Pelanggan Terdaftar:** Saya ingin melihat daftar seluruh riwayat pembelian saya, mengunduh faktur resmi (*invoice*), dan melihat nomor resi pengiriman kurir.
- **Sebagai Penerima Perhiasan:** Saya ingin menerima notifikasi email otomatis setiap kali status pesanan saya berubah, terutama saat pesanan telah diserahkan ke pihak kurir berasuransi beserta nomor pelacakan resi.
- **Sebagai Staf Pemenuhan (Admin/Ops):** Saya ingin memperbarui status pesanan dari `paid` ke `processing`, memasukkan catatan nomor resi pengiriman, dan menandai pesanan sebagai `completed` setelah tanda terima kurir terkonfirmasi.

---

## 3. Aturan Bisnis (Business Rules)

1. **Siklus Hidup Status Pesanan (Order Status Lifecycle):**
   - Transisi status bersifat searah dan deterministik:
     - `pending_payment` → Menunggu pelunasan pembayaran.
     - `paid` → Pembayaran lunas dan diverifikasi (otomatis via webhook atau manual via admin).
     - `processing` → Perhiasan sedang dipersiapkan di workshop (inspeksi mutu, penyesuaian ukuran, pengemasan eksklusif).
     - `shipped` → Paket telah diserahkan ke jasa kurir khusus berasuransi perhiasan; nomor resi wajib diisi.
     - `completed` → Paket telah diterima oleh pelanggan dengan aman.
   - Status Terminal / Pengecualian:
     - `cancelled` → Pesanan dibatalkan (misal: pembayaran kedaluwarsa atau pembatalan sah sebelum diproses).
     - `refunded` → Dana dikembalikan kepada pelanggan (memerlukan otorisasi `super_admin`).
2. **Keamanan Pelacakan Tamu (Guest Lookup Security):**
   - Pelacakan tamu mewajibkan pencocokan ganda (*dual match*): `order_number` persis DAN `guest_email` (case-insensitive).
   - Sistem tidak boleh menampilkan detail alamat lengkap penerima pada halaman pelacakan publik (gunakan penyamaran/masking: misal `Jl. Sen***, Jakarta Selatan`).
   - Menerapkan pembatasan frekuensi (*rate limiting*) 10 percobaan pencarian per menit per IP untuk mencegah *brute-force enumerations*.
3. **Pencatatan Riwayat Snapshot:**
   - Data produk pada baris pesanan (`order_lines`) adalah salinan permanen (*snapshot*) harga dan nama produk pada waktu transaksi dibuat, sehingga perubahan harga atau katalog di masa depan tidak mempengaruhi data historis pesanan.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini berpusat pada tabel:

- `orders`: Data kepala pesanan (`id`, `order_number`, `user_id`, `guest_email`, `status`, `currency`, `subtotal_minor`, `shipping_minor`, `total_minor`, `shipping_address_id`, `tracking_number`, `courier_name`, `notes`, `created_at`, `updated_at`).
- `order_lines`: Rincian item pesanan (`id`, `order_id`, `piece_id`, `variant_id`, `quantity`, `unit_price_minor_idr`, `unit_price_minor_usd`).
- `pieces` & `piece_variants`: Referensi informasi karya dan ukuran varian.
- `addresses`: Alamat tujuan pengiriman fisik.
- `payments`: Riwayat pembayaran yang terkait dengan pesanan.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### 1. Pelacakan Pesanan Tamu

- **`POST /api/orders/track`**
  - **Request Body:**
    ```json
    {
      "orderNumber": "BJ-20261011-8F3A",
      "email": "customer@example.com"
    }
    ```
  - **Respon 200 OK:**
    ```json
    {
      "success": true,
      "data": {
        "orderNumber": "BJ-20261011-8F3A",
        "createdAt": "2026-10-11T14:30:00Z",
        "status": "processing",
        "currency": "IDR",
        "totalMinor": 18550000000,
        "items": [
          {
            "name": "Celestial Solitaire Diamond Ring",
            "ringSize": "6.0",
            "quantity": 1,
            "unitPriceMinor": 18500000000
          }
        ],
        "shipping": {
          "courier": "Bifrost Armoured Courier / RPX Valuables",
          "trackingNumber": "RPX-SEC-998822",
          "maskedAddress": "Jl. Sen***, Kebayoran Baru, Jakarta Selatan"
        }
      }
    }
    ```
  - **Respon 404 Not Found:**
    ```json
    {
      "success": false,
      "error": {
        "code": "ORDER_NOT_FOUND",
        "message": "Pesanan dengan nomor dan email tersebut tidak ditemukan. Mohon periksa kembali input Anda."
      }
    }
    ```

### 2. Server Action Manajemen Pesanan Admin

- **`updateOrderStatusAction({ orderId: string, status: OrderStatus, trackingNumber?: string, courierName?: string })`**
  - Terproteksi peran `admin` dan `super_admin`. Memicu pengiriman email status update ke email pemesan via Resend.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `OrderTrackingForm`: Formulir masukan nomor pesanan dan email dengan gaya editorial minimalis.
- `OrderTimeline`: Visualisasi garis waktu tahapan pesanan (Dipesan → Pembayaran Terverifikasi → Pengerjaan Workshop → Dikirim → Selesai) menggunakan tipografi tenang dan status indikator warna `--ql-gold-deep` atau `--ql-ink`.
- `OrderInvoiceView`: Tata letak faktur digital resmi beresolusi cetak rapi yang mencantumkan detail nomor sertifikat perhiasan, rincian biaya, dan kebijakan garansi.
- `TrackingBadge`: Lencana status pesanan dengan gaya warna semantik tenang (tidak mencolok/norak).

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Pengguna tamu dapat melihat status pesanan hanya jika kombinasi nomor pesanan dan email cocok secara akurat.
- [ ] Pengguna terdaftar yang login dapat melihat seluruh daftar pesanannya di `/account/orders` tanpa memasukkan email kembali.
- [ ] Alamat pengiriman pada halaman pelacakan tamu disamarkan demi privasi data pelanggan.
- [ ] Setiap pembaruan status pesanan oleh admin tercatat di basis data dan mengirimkan email notifikasi ke pelanggan melalui Resend.
- [ ] Upaya pelacakan berulang yang gagal dibatasi oleh rate limiter pada 10 request per menit.
- [ ] Rincian harga pada faktur pesanan tetap mempertahankan snapshot historis saat transaksi terjadi.

---

## 8. Dokumen Terkait

- [CART_CHECKOUT.md](./CART_CHECKOUT.md) — Alur pembentukan awal pesanan
- [PAYMENT.md](./PAYMENT.md) — Verifikasi pembayaran yang mengubah status ke `paid`
- [ACCOUNT.md](./ACCOUNT.md) — Tampilan riwayat pesanan pada dashboard pengguna
