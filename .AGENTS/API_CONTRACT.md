# API_CONTRACT.md — Kontrak Endpoint API & Protokol Webhook

> **Tujuan:** Dokumen ini mendefinisikan spesifikasi antarmuka jaringan HTTP (Route Handlers) dan protokol webhook untuk sistem Bifrost Jewellery. Dokumen ini menetapkan format amplop (*envelope*) response standar, kontrak payload request/response, penanganan status code, proteksi rate limiting, serta penegasan bahwa seluruh mutasi data internal admin dilakukan melalui Server Actions.

---

## 1. Konfigurasi Dasar Jaringan (Network Configuration)

- **Base URL (Development):** `http://localhost:3000`
- **Base URL (Production):** `<TBD: domain>`
- **Routing Pattern:** RESTful Route Handlers di bawah `src/app/api/` (tanpa prefix `/v1` sesuai konvensi Next.js App Router).
- **Format Header Standar:**
  - `Content-Type: application/json`
  - `Accept: application/json`
- **Mekanisme Autentikasi:**
  - Endpoint Publik: Tidak memerlukan token autentikasi.
  - Endpoint Webhook Midtrans: Verifikasi SHA-512 signature key melalui header dan payload body.
  - Mutasi Admin: Menggunakan **Server Actions** terproteksi sesi Auth.js v5 cookie, bukan REST API publik.

---

## 2. Format Amplop Response Standar (Standard Response Envelopes)

Semua Route Handler publik WAJIB mengembalikan respon yang dibungkus dalam format amplop terstruktur berikut:

### Format Respon Sukses (Success Envelope)
```json
{
  "success": true,
  "data": {
    "items": []
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "totalItems": 100,
    "totalPages": 5
  }
}
```

### Format Respon Kegagalan (Error Envelope)
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input data tidak valid sesuai skema Zod.",
    "details": {
      "carat_min": "Nilai karat minimal harus berupa angka positif."
    }
  }
}
```

---

## 3. Spesifikasi Endpoint Publik & Webhook

### 3.1. `GET /api/pieces` — Daftar & Filter Perhiasan
Mengambil daftar perhiasan aktif dengan dukungan filter parameter 4Cs berlian, kategori, dan pengurutan.

- **Autentikasi:** Publik
- **Query Parameters:**
  - `category` (string, opsional): Slug kategori (misal: `rings`)
  - `collection` (string, opsional): Slug koleksi
  - `origin` (enum: `natural` | `lab_grown`, opsional): Asal berlian
  - `shape` (enum: `round` | `oval` | `emerald` | `pear` | `princess` | `cushion`, opsional)
  - `carat_min` (float, opsional): Berat karat minimal
  - `carat_max` (float, opsional): Berat karat maksimal
  - `price_max_idr` (integer, opsional): Batas harga atas dalam IDR minor unit
  - `availability` (enum: `in_stock` | `made_to_order`, opsional)
  - `sort` (enum: `price_asc` | `price_desc` | `newest` | `carat_desc`, default: `newest`)
  - `page` (integer, default: 1)
  - `limit` (integer, default: 20)
- **Response:**
  - `200 OK`: Mengembalikan array perhiasan beserta data gambar primer, spesifikasi logam, dan ringkasan 4Cs berlian.
  - `400 Bad Request`: Parameter query tidak valid.

---

### 3.2. `GET /api/pieces/[slug]` — Detail Perhiasan Spesifik
Mengambil data detail lengkap satu karya perhiasan berdasarkan slug URL.

- **Autentikasi:** Publik
- **Path Parameter:** `slug` (string, wajib)
- **Response:**
  - `200 OK`: Mengembalikan detail perhiasan lengkap mencakup galeri gambar, varian ukuran cincin dan stoknya, spesifikasi 4Cs berlian lengkap, serta nomor dan tautan sertifikat grading GIA/IGI.
  - `404 Not Found`: Perhiasan tidak ditemukan atau berstatus `is_published: false`.

---

### 3.3. `POST /api/checkout` — Inisialisasi Transaksi & Menerbitkan Snap Token
Menerima rincian pembelian, melakukan perhitungan ulang harga di sisi server secara mutlak, membuat pesanan dengan status `pending_payment`, dan menerbitkan token pembayaran Midtrans Snap.

- **Autentikasi:** Publik (Mendukung Guest & Logged-in User)
- **Request Body:**
  ```json
  {
    "cartLines": [
      {
        "pieceId": "a2c3d4e5-f6a7-4b8c-9d0e-1f2a3b4c5d6e",
        "variantId": "b3c4d5e6-f7a8-4c9d-0e1f-2a3b4c5d6e7f",
        "quantity": 1
      }
    ],
    "currency": "IDR",
    "guestEmail": "concierge.client@example.com",
    "guestName": "Eleanor Vance",
    "guestPhone": "+6281234567890",
    "shippingAddress": {
      "recipientName": "Eleanor Vance",
      "phone": "+6281234567890",
      "addressLine1": "Jl. Senopati No. 45",
      "addressLine2": "Apartemen District 8 Tower Infinity",
      "city": "Jakarta Selatan",
      "province": "DKI Jakarta",
      "postalCode": "12190"
    },
    "notes": "Complimentary engraving: E & V 2026"
  }
  ```
- **Logika Server:**
  1. Validasi Zod pada seluruh field request.
  2. Query database untuk mengambil harga resmi (`price_idr_minor` / `price_usd_minor`) dari setiap `pieceId`.
  3. Verifikasi ketersediaan stok fisik jika varian memiliki stok terbatas.
  4. Hitung subtotal, asuransi kirim, dan total tagihan mutlak di server.
  5. Buat entitas `orders` (`status: pending_payment`) dan baris `order_lines`.
  6. Hubungi Midtrans API untuk menerbitkan `snap_token`.
  7. Simpan entitas `payments` (`method: midtrans_snap`, `status: pending`, `provider_ref: snap_token`).
- **Response:**
  - `201 Created`:
    ```json
    {
      "success": true,
      "data": {
        "orderNumber": "BJ-20261011-8821",
        "snapToken": "d41d8cd98f00b204e9800998ecf8427e",
        "redirectUrl": "https://app.sandbox.midtrans.com/snap/v2/vtweb/d41d8cd98f00b204e9800998ecf8427e"
      }
    }
    ```
  - `400 Bad Request`: Skema data pemesan atau alamat tidak lengkap.
  - `409 Conflict`: Salah satu karya perhiasan telah dipesan oleh pelanggan lain (*out of stock*).

---

### 3.4. `POST /api/payments/midtrans/webhook` — Webhook Notifikasi Midtrans
Menerima notifikasi asinkron dari server Midtrans mengenai status penyelesaian transaksi pembayaran.

- **Autentikasi:** Verifikasi tanda tangan SHA-512 `signature_key`
- **Request Body (Standar Notifikasi Midtrans):**
  ```json
  {
    "order_id": "BJ-20261011-8821",
    "status_code": "200",
    "gross_amount": "15000000.00",
    "signature_key": "abc123...sha512",
    "transaction_status": "settlement",
    "payment_type": "bank_transfer"
  }
  ```
- **Logika Server:**
  1. Hitung `expected_signature = SHA512(order_id + status_code + gross_amount + MIDTRANS_SERVER_KEY)`.
  2. Jika signature tidak cocok, kembalikan `401 Unauthorized`.
  3. Periksa status pesanan di database (pemeriksaan idempotensi). Jika pesanan sudah berstatus `paid`, segera kembalikan `200 OK` tanpa memicu ulang pengiriman email.
  4. Jika status `settlement` atau `capture` (dengan `fraud_status: accept`):
     - Ubah `payments.status` menjadi `paid`.
     - Ubah `orders.status` menjadi `paid`.
     - Picu pengiriman email konfirmasi pesanan resmi ke pelanggan via Resend.
  5. Jika status `expire` atau `deny`, ubah status order menjadi `cancelled`.
- **Response:**
  - `200 OK`: `{ "success": true, "message": "Notification handled successfully" }`
  - `401 Unauthorized`: Signature key tidak valid.

---

### 3.5. `POST /api/payments/manual-transfer` — Pengajuan Bukti Transfer Bank Manual
Menerima konfirmasi dan unggahan URL bukti transfer bank manual dari pelanggan untuk transaksi bernilai tinggi.

- **Autentikasi:** Publik / Pelanggan
- **Request Body:**
  ```json
  {
    "orderNumber": "BJ-20261011-8821",
    "senderBankName": "BCA",
    "senderAccountName": "Eleanor Vance",
    "proofImageUrl": "https://...blob.vercel-storage.com/proofs/receipt-123.jpg",
    "amountPaidMinor": 15000000
  }
  ```
- **Response:**
  - `201 Created`: Status pembayaran diperbarui menjadi `awaiting_verification`.
  - `400 Bad Request`: Nomor pesanan tidak ditemukan atau jumlah transfer tidak cocok.

---

### 3.6. `POST /api/appointments` — Reservasi Janji Temu Butik
Mencatat jadwal pemesanan sesi konsultasi tatap muka privat di butik fisik.

- **Autentikasi:** Publik
- **Request Body:**
  ```json
  {
    "boutiqueId": "c4d5e6f7-a8b9-4c0d-1e2f-3a4b5c6d7e8f",
    "scheduledAt": "2026-10-15T14:00:00.000Z",
    "guestName": "Julian Croft",
    "guestEmail": "julian.croft@example.com",
    "guestPhone": "+6281987654321",
    "pieceId": "a2c3d4e5-f6a7-4b8c-9d0e-1f2a3b4c5d6e",
    "notes": "Looking for a 2-carat oval natural diamond engagement ring."
  }
  ```
- **Response:**
  - `201 Created`: Reservasi berhasil tercatat dengan status `requested`; memicu email notifikasi ke tamu dan staf butik.
  - `400 Bad Request`: Format tanggal atau email tidak valid.
  - `409 Conflict`: Slot waktu pada butik tersebut telah terisi penuh.

---

### 3.7. `GET /api/appointments/slots` — Ketersediaan Slot Waktu Butik
Mengambil daftar slot waktu janji temu yang masih tersedia untuk tanggal dan butik tertentu.

- **Autentikasi:** Publik
- **Query Parameters:**
  - `boutiqueId` (UUID, wajib)
  - `date` (format YYYY-MM-DD, wajib)
- **Response:**
  - `200 OK`: Mengembalikan daftar waktu (misal: `["11:00", "13:00", "15:00", "17:00"]`) yang belum terisi.

---

### 3.8. `POST /api/inquiries` — Pengajuan Formulir Konsultasi
Menerima permohonan konsultasi perhiasan kustom (*bespoke inquiry*).

- **Autentikasi:** Publik
- **Request Body:** `{ name, email, phone, pieceId?, message }`
- **Response:**
  - `201 Created`: Berhasil disimpan dengan status `new`.

---

### 3.9. `POST /api/auth/[...nextauth]` — Handler Autentikasi Auth.js v5
Menangani siklus login sesi credentials admin dan otentikasi pelanggan.

---

## 4. Penegasan Mutasi Admin via Server Actions

> ⚠️ **Catatan Arsitektur:** Seluruh operasi penambahan, pengubahan, dan penghapusan data di panel admin (seperti pembuatan produk, pengubahan inventori, persetujuan verifikasi transfer bank, dan perubahan status janji temu) **TIDAK MENGGUNAKAN REST API**, melainkan menggunakan **Server Actions** terproteksi sesi yang divalidasi dengan Zod di `src/lib/actions/`.

---

## 5. Tabel Kode Status HTTP (HTTP Status Codes)

| Kode HTTP | Status Standar | Konteks Penggunaan dalam Sistem |
|---|---|---|
| `200 OK` | Sukses | Request GET berhasil mengembalikan data atau webhook diproses. |
| `201 Created` | Dibuat | Berhasil membuat pesanan checkout, janji temu, atau inquiry baru. |
| `400 Bad Request` | Permintaan Buruk | Payload JSON tidak sesuai skema validasi Zod. |
| `401 Unauthorized` | Belum Diotorisasi | Signature webhook salah atau sesi admin tidak ditemukan. |
| `403 Forbidden` | Dilarang | Pengguna tidak memiliki hak akses role yang memadai. |
| `404 Not Found` | Tidak Ditemukan | Perhiasan dengan slug tertentu atau pesanan tidak ada di database. |
| `409 Conflict` | Konflik Status | Slot janji temu sudah terisi atau perhiasan telah dibeli orang lain. |
| `429 Too Many Requests`| Batas Terlampaui | Melebihi batas kuota pemanggilan rate limiting. |
| `500 Internal Error` | Kesalahan Server | Kegagalan sistem internal yang tidak tertangani. |

---

## 6. Kebijakan Pembatasan Frekuensi (Rate Limiting)

Untuk menjaga stabilitas sistem dan mencegah eksploitasi, pembatasan laju pemanggilan (*rate limiting*) berbasis IP diterapkan:
- **Autentikasi Admin (`/api/auth/**`):** Maksimal 5 permintaan / menit / IP (Pencegahan brute-force).
- **Booking Janji Temu (`POST /api/appointments`):** Maksimal 10 permintaan / menit / IP.
- **Formulir Konsultasi (`POST /api/inquiries`):** Maksimal 5 permintaan / menit / IP.
- **Kueri Katalog Publik (`GET /api/pieces`):** Maksimal 120 permintaan / menit / IP.

---

## 7. Dokumen Terkait

- [DATABASE.md](./DATABASE.md) — Struktur data dan konvensi kolom database
- [ARCHITECTURE.md](./ARCHITECTURE.md) — Alur sequence diagram alur checkout
- [CODING_RULES.md](./CODING_RULES.md) — Pola implementasi Server Actions
- [GLOSSARY.md](./GLOSSARY.md) — Definisi status pesanan dan pembayaran
