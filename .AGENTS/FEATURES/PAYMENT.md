# 💳 FEATURE — Gerbang Pembayaran & Transfer Manual (PAYMENT)

> **Status:** Planned  
> **Scope:** Payment Processing (Midtrans Gateway + Manual Bank Transfer + Webhooks)  
> **Primary Goal:** Memproses transaksi keuangan secara aman, patuh regulasi, dan fleksibel melalui dua kanal utama: pembayaran digital instan via Midtrans Snap (Kartu Kredit/Debit, Virtual Account, QRIS) serta transfer bank manual dengan verifikasi staf admin, didukung verifikasi tanda tangan digital (signature hash) dan mekanisme idempoten mutlak.

---

## 1. Ringkasan (Overview)

Pembelian perhiasan mewah menuntut tingkat keandalan dan keamanan transaksi tertinggi. Modul Pembayaran Bifrost Jewellery menggabungkan kemudahan gerbang pembayaran digital berstandar PCI-DSS dengan opsi tradisional perbankan lokal yang sering kali dipilih untuk nominal transaksi tinggi (High-Value Transactions).

Sistem pembayaran dirancang dengan dua pilar utama:
1. **Midtrans Snap:** Pelanggan menyelesaikan pembayaran langsung melalui antarmuka modal Snap yang terisolasi. Notifikasi status pembayaran diterima secara asinkron melalui webhook terenkripsi server-ke-server (*server-to-server HTTP POST*).
2. **Transfer Bank Manual:** Pelanggan mentransfer dana langsung ke rekening koran resmi butik Bifrost Jewellery dan mengunggah referensi transfer/bukti bayar. Pesanan beralih ke status `awaiting_verification` hingga diverifikasi dan disetujui secara manual oleh staf keuangan pada panel admin.

Seluruh data transaksi dilindungi oleh verifikasi tanda tangan kriptografi SHA-512 dan kunci idempoten (*idempotency keys*) untuk mencegah pembayaran ganda atau *replay attack*.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pembeli:** Saya ingin membayar pesanan saya menggunakan metode pembayaran digital modern (Kartu Kredit internasional, BCA/Mandiri Virtual Account, atau QRIS) dengan instruksi langkah demi langkah yang jelas.
- **Sebagai Pembeli Transaksi Besar:** Saya ingin memilih opsi transfer bank langsung ke rekening resmi perusahaan untuk menghindari limitasi batas harian kartu kredit.
- **Sebagai Pembeli yang Sudah Membayar:** Saya ingin menerima email konfirmasi pembayaran resmi seketika setelah pembayaran saya divalidasi oleh gerbang pembayaran.
- **Sebagai Staf Admin Keuangan:** Saya ingin melihat daftar pembayaran manual yang membutuhkan verifikasi, memeriksa mutasi bank, dan menyetujui atau menolak bukti transfer dengan pencatatan audit lengkap.

---

## 3. Aturan Bisnis (Business Rules)

1. **Keamanan PCI-DSS Terisolasi:**
   - Aplikasi Bifrost Jewellery tidak pernah menyimpan, membaca, atau memproses nomor kartu kredit mentah maupun kode CVV. Seluruh proses penanganan data kartu dikelola secara eksklusif oleh Midtrans Snap (*Hosted Payment Gateway*).
2. **Kondisi dan Transisi Status Pembayaran:**
   - Status pembayaran (`payments.status`) memiliki alur deterministik:
     - Midtrans: `pending` → `paid` (sukses) atau `failed` / `expired` (gagal/kedaluwarsa).
     - Transfer Manual: `pending` → `awaiting_verification` (bukti diunggah) → `paid` (disetujui admin) atau `failed` (ditolak).
3. **Verifikasi Tanda Tangan Webhook (SHA-512 Signature Hash):**
   - Setiap webhook dari Midtrans wajib diverifikasi integritasnya sebelum data diproses.
   - Hash SHA-512 dihitung dari formula:
     `SHA512(order_id + status_code + gross_amount + ServerKey)`
   - Jika hash yang dihitung tidak sama dengan `signature_key` yang dikirim oleh Midtrans, permintaan ditolak dengan kode `401 Unauthorized`.
4. **Idempotensi Pemrosesan Webhook (Absolute Idempotency):**
   - Midtrans dapat mengirimkan notifikasi berulang untuk transaksi yang sama.
   - Server wajib memeriksa apakah status transaksi pada basis data sudah berada pada status final (`paid`). Jika sudah, server langsung mengembalikan respon `200 OK` tanpa memicu ulang pengiriman email atau mutasi data.
5. **Batas Waktu Kedaluwarsa:**
   - Transaksi transfer manual yang tidak melampirkan bukti dalam waktu 24 jam otomatis beralih menjadi `expired` dan stok barang dikembalikan.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini beroperasi pada tabel-tabel berikut:

- `payments`: Rekaman transaksi pembayaran (`id`, `order_id`, `method`, `status`, `provider_ref`, `amount_minor`, `verified_by`, `verified_at`, `created_at`, `updated_at`).
  - Enum `method`: `midtrans_snap`, `manual_transfer`.
  - Enum `status`: `pending`, `awaiting_verification`, `paid`, `failed`, `expired`.
- `orders`: Entitas pesanan yang diperbarui statusnya (`status`: `pending_payment` → `paid`).
- `users`: Relasi akun admin yang melakukan verifikasi manual (`verified_by`).

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### 1. Midtrans Webhook Handler

- **`POST /api/payments/midtrans/webhook`**
  - **Headers:** `Content-Type: application/json`
  - **Payload dari Midtrans:**
    ```json
    {
      "transaction_time": "2026-10-11 14:32:00",
      "transaction_status": "settlement",
      "transaction_id": "midtrans-tx-998877",
      "status_message": "Midtrans payment notification",
      "status_code": "200",
      "signature_key": "a1b2c3d4e5f6...computed-sha512-hash...",
      "payment_type": "credit_card",
      "order_id": "BJ-20261011-8F3A",
      "gross_amount": "18550000.00",
      "fraud_status": "accept",
      "currency": "IDR"
    }
    ```
  - **Respon:**
    - `200 OK`: Notifikasi berhasil diproses atau sudah berstatus final (idempoten).
    - `401 Unauthorized`: Tanda tangan hash SHA-512 tidak cocok dengan secret key server.
    - `404 Not Found`: `order_id` tidak ditemukan dalam basis data.

### 2. Konfirmasi Transfer Bank Manual

- **`POST /api/payments/manual-transfer`**
  - **Request Body:**
    ```json
    {
      "orderNumber": "BJ-20261011-8F3A",
      "senderBank": "BCA",
      "senderAccountName": "Jane Doe",
      "senderAccountNumber": "0123456789",
      "transferAmountMinor": 18550000000,
      "proofFileUrl": "https://blob.bifrostjewellery.com/proofs/proof-8f3a.jpg"
    }
    ```
  - **Respon 200 OK:**
    ```json
    {
      "success": true,
      "message": "Bukti transfer berhasil diterima. Tim keuangan kami akan memverifikasi dalam waktu maksimal 1x24 jam kerja."
    }
    ```

### 3. Server Action Verifikasi Admin

- **`verifyManualPaymentAction({ paymentId: string, action: 'approve' | 'reject', notes?: string })`**
  - Hanya dapat dieksekusi oleh pengguna berstatus role `admin` atau `super_admin`.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `MidtransSnapModal`: Modal bawaan Midtrans yang dipicu oleh JavaScript SDK Snap melalui token sesi.
- `ManualTransferInstructionCard`: Komponen kartu elegan yang mencantumkan:
  - Nomor Rekening Perusahaan: `PT Bifrost Mahakarya Mulia` (BCA / Mandiri).
  - Jumlah Tepat Transfer: Nominal rupiah hingga digit unik (jika ada).
  - Tombol Salin Cepat (*Quick Copy*) untuk nomor rekening dan nominal.
- `PaymentProofUploader`: Komponen seret-dan-lepas (*drag-and-drop*) untuk mengunggah berkas foto bukti transfer (JPG, PNG, PDF maks. 5MB) ke Vercel Blob.
- `PaymentStatusBanner`: Banner status di halaman rincian pesanan (misal: "Menunggu Verifikasi Pembayaran" atau "Pembayaran Terkonfirmasi").

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Webhook Midtrans dengan status `settlement` atau `capture` (fraud_status: `accept`) memperbarui status `payments` menjadi `paid` dan status `orders` menjadi `paid` dalam satu transaksi basis data.
- [ ] Webhook dengan tanda tangan SHA-512 yang tidak valid ditolak dengan kode status 401 dan tidak mengubah data apa pun.
- [ ] Panggilan webhook duplikat untuk pesanan yang telah lunas diproses secara idempoten: mengembalikan 200 OK tanpa mengirim email ganda.
- [ ] Form transfer manual memvalidasi ukuran berkas bukti (maks. 5MB) dan format gambar sebelum mengunggah.
- [ ] Staf admin yang menyetujui bukti transfer melalui Server Action berhasil mengubah status pesanan ke `paid` dan merekam ID admin serta timestamp verifikasi.
- [ ] Email notifikasi bukti pembayaran terkirim secara otomatis kepada pelanggan saat status beralih ke `paid`.

---

## 8. Dokumen Terkait

- [CART_CHECKOUT.md](./CART_CHECKOUT.md) — Pembuatan order dan inisiasi token pembayaran
- [ORDERS.md](./ORDERS.md) — Siklus hidup status pesanan pasca-pembayaran
- [PAYMENT_WEBHOOK_PATTERN.md](../PATTERNS/PAYMENT_WEBHOOK_PATTERN.md) — Pola implementasi teknis webhook handler
