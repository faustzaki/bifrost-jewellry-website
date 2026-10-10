# 🛡️ FLOW — Alur Kerja Operasional & Panel Admin (ADMIN_FLOW)

> **Aktor:** Staf Admin, Super Admin, Sistem Internal Next.js, Vercel Blob Storage  
> **Cakupan:** Otentikasi Admin & Proteksi Sesi → Dashboard Metrik Butik → Pengelolaan Katalog & Spesifikasi Berlian 4Cs → Verifikasi Pembayaran Manual → Pengelolaan Janji Temu & Inquiries.

---

## 1. Diagram Alur (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor Admin as Staf Admin / Super Admin
    participant UI as Peramban Admin (/admin)
    participant Auth as Auth.js v5
    participant Server as Next.js Server Actions
    participant DB as PostgreSQL
    participant Blob as Vercel Blob Storage

    Admin->>UI: Akses /admin/login
    UI->>Auth: Kirim Kredensial Email & Kata Sandi
    Auth->>DB: Validasi Hash Password & Cek Role (admin/super_admin)
    DB--Auth: Peran Terverifikasi
    Auth-->>UI: Terbitkan Cookie Sesi HttpOnly Terenkripsi

    Admin->>UI: Buka Dashboard (/admin)
    UI->>Server: Muat Data Ringkasan Metrik (RSC)
    Server->>DB: Kueri Pesanan Pending, Janji Temu Hari Ini, Inquiry Baru
    DB--Server: Data Agregat
    Server-->>UI: Render Dasbor Operasional

    alt Manajemen Katalog Perhiasan
        Admin->>UI: Buka /admin/pieces/new (Tambah Karya Baru)
        Admin->>UI: Unggah Foto Studio & Isi 4Cs Berlian (GIA/IGI)
        UI->>Blob: Upload Gambar Studio Teroptimasi
        Blob-->>UI: Return image_url
        Admin->>UI: Klik "Save & Publish Piece"
        UI->>Server: Server Action createPieceAction(data)
        Server->>DB: Transaksi Atomik: Simpan piece, diamonds, piece_variants
        Server->>Server: revalidatePath('/collections')
        Server-->>UI: Berhasil & Redirect ke Daftar Katalog
    else Verifikasi Transfer Bank Manual
        Admin->>UI: Buka /admin/orders?status=awaiting_verification
        UI-->>Admin: Tampilkan Foto Bukti Transfer & Jumlah Pesanan
        Admin->>Admin: Cek Mutasi Rekening Bank Perusahaan
        Admin->>UI: Klik "Approve Payment"
        UI->>Server: Server Action approveManualPaymentAction(paymentId)
        Server->>DB: Transaksi: Update payments (paid) & orders (paid), catat verified_by
        Server-->>UI: Status Diperbarui & Memicu Notifikasi Pelanggan
    else Koordinasi Janji Temu & Inquiry
        Admin->>UI: Buka /admin/appointments (Lihat Agenda Kunjungan)
        Admin->>UI: Tandai Janji Temu Selesai ("completed")
        Admin->>UI: Buka /admin/inquiries & Balas Konsultasi Klien Kustom
        UI->>Server: Server Action updateInquiryStatusAction(inquiryId, 'contacted')
        Server->>DB: Update Status Inquiry
    end
```

---

## 2. Tabel Langkah Pengguna & Sistem

| Langkah | Aksi Admin (Admin Action) | Respon Sistem (System Action) | State / Data Terkait |
|:---:|---|---|---|
| **1** | Mengakses `/admin` dan memasukkan kredensial login staf. | Auth.js memverifikasi kredensial. Jika peran pengguna adalah `admin` atau `super_admin`, sistem mengizinkan akses dan menerbitkan cookie sesi terenkripsi. | Sesi admin terotentikasi. |
| **2** | Membuka Dashboard Utama `/admin`. | Server Component mengambil data metrik real-time: jumlah pesanan baru, total nominal pembayaran menunggu verifikasi, dan janji temu butik hari ini. | Tampilan metrik ter-render. |
| **3** | Menavigasi ke menu Katalog (`/admin/pieces`). | Sistem menampilkan tabel data seluruh karya perhiasan beserta status publikasi, jenis logam, dan spesifikasi karat berlian. | Data katalog termuat dengan pagination server-side. |
| **4** | Menambah karya baru dan mengunggah foto produk studio. | Gambar diunggah langsung ke Vercel Blob Storage melalui token aman, menghasilkan URL berkas publik permanen berformat WebP. | Foto tersimpan di Blob. |
| **5** | Mengisi spesifikasi 4Cs berlian (Origin, Carat, Cut, Color, Clarity, No. Sertifikat GIA/IGI) dan menyimpan karya. | Server Action `createPieceAction` memvalidasi skema Zod, mengeksekusi operasi transaksi Prisma (`pieces`, `diamonds`, `piece_variants`), dan memanggil `revalidatePath`. | Data tersimpan atomik di DB; cache ISR publik di-refresh. |
| **6** | Menavigasi ke menu Pesanan (`/admin/orders`) untuk meninjau pembayaran manual. | Sistem menyajikan daftar transaksi dengan status `awaiting_verification` beserta pratinjau foto bukti transfer. | Antarmuka verifikasi aktif. |
| **7** | Mengklik tombol "Approve Payment" setelah memeriksa mutasi rekening bank. | Server Action `approveManualPaymentAction` memverifikasi izin admin, memperbarui status pesanan menjadi `paid`, merekam `verified_by = admin.id`, dan memicu email tanda terima ke pelanggan. | Status order: `paid`, audit log tercatat. |
| **8** | Meninjau janji temu butik harian dan pesan konsultasi concierge. | Admin memperbarui status janji temu menjadi `completed` atau memperbarui tiket inquiry menjadi `contacted` setelah menghubungi klien via WhatsApp/telepon. | Status janji temu & inquiry terbarui. |

---

## 3. Penanganan Error & Kasus Tepi (Edge Cases)

| Skenario Kasus Tepi | Gejala / Kondisi | Solusi & Penanganan Sistem |
|---|---|---|
| **Sesi Admin Kedaluwarsa di Tengah Pengeditan** | Admin membiarkan formulir terbuka selama berjam-jam lalu menekan simpan. | Server Action menolak mutasi dengan `UnauthorizedError`. Klien menampilkan modal pemberitahuan sesi habis dan meminta login ulang tanpa menghapus data input formulir di peramban (*form state preservation*). |
| **Unggahan Berkas Terlalu Besar / Format Salah** | Staf mencoba mengunggah berkas foto RAW kamera berukuran 50MB. | Klien dan server menolak berkas dengan pesan validasi "Ukuran foto maksimal 10MB dalam format WebP, JPG, atau PNG". |
| **Nomor Sertifikat Berlian Duplikat** | Admin tidak sengaja memasukkan nomor sertifikat GIA yang telah terdaftar pada karya lain. | Basis data menegakkan aturan unik `UNIQUE(certificate_number)`. Server Action menangkap *Prisma P2002 Unique Constraint Violation* dan menampilkan pesan kesalahan ramah "Nomor sertifikat GIA tersebut telah digunakan pada karya lain." |
| **Penolakan Bukti Transfer Bank (Reject)** | Bukti transfer palsu atau dana belum masuk rekening koran. | Admin menekan tombol "Reject Payment" dengan menyertakan alasan penolakan. Status pembayaran beralih menjadi `failed` dan sistem mengirimkan email klarifikasi sopan ke pelanggan. |
| **Akses Modul Pengguna oleh Staf Biasa** | Admin dengan peran `admin` mencoba membuka rute `/admin/users` yang khusus `super_admin`. | Server Component mendeteksi peran non-super-admin dan merender tampilan 403 Forbidden (*Akses Ditolak: Fitur hanya dapat dikelola oleh Super Admin*). |
