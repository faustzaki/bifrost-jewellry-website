# 🏛️ FLOW — Alur Pemesanan Janji Temu Butik (APPOINTMENT_FLOW)

> **Aktor:** Calon Klien Butik (*Guest* / *Customer*), Sistem Penjadwalan, Layanan Surel Resend, Staf Concierge Butik  
> **Cakupan:** Pemilihan Lokasi Butik → Pengecekan Slot Waktu Bebas Konflik → Pengisian Preferensi & Karya → Konfirmasi Instan & Kalender (.ics) → Pengingat Kunjungan & Sesi Tatap Muka.

---

## 1. Diagram Alur (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor Client as Calon Klien
    participant UI as Peramban Web
    participant Server as Server (Next.js API)
    participant DB as PostgreSQL
    participant Email as Layanan Email (Resend)
    actor Concierge as Staf Butik / Concierge

    Client->>UI: Buka /appointments (atau klik "Book Viewing" dari PDP)
    UI->>Server: GET /api/boutiques (Ambil daftar butik aktif)
    Server->>DB: Query Boutiques WHERE is_active = true
    DB--Server: Data Butik Fisik
    Server-->>UI: Tampilkan Pilihan Butik Fisik

    Client->>UI: Pilih Butik & Pilih Tanggal Kunjungan (misal: 15 Okt 2026)
    UI->>Server: GET /api/appointments/slots?boutiqueId={id}&date=2026-10-15
    Server->>DB: Cek Janji Temu Terdaftar pada Tanggal & Butik Tersebut
    DB--Server: Slot Waktu yang Sudah Terisi
    Server-->>UI: Kembalikan Daftar Slot Jam (Tersedia vs Terisi)
    UI-->>Client: Render Grid Jam 60 Menit yang Dapat Dipilih

    Client->>UI: Pilih Jam (misal: 15:00) & Isi Data Diri + Catatan Karya
    Client->>UI: Klik "Confirm Private Appointment"
    UI->>Server: POST /api/appointments (Payload data booking)

    Note over Server,DB: Validasi Zod & Pengecekan Konflik Atomik (409)
    Server->>DB: SELECT FOR UPDATE / Cek Slot 15:00 Masih Kosong
    alt Slot Sudah Diambil Tamu Lain
        DB--Server: Konflik Terdeteksi
        Server-->>UI: 409 Conflict (Pesan: Slot tidak lagi tersedia)
        UI-->>Client: Tampilkan Notifikasi & Minta Pilih Jam Lain
    else Slot Masih Kosong (Sukses)
        Server->>DB: INSERT INTO appointments (status: confirmed)
        DB--Server: Janji Temu Berhasil Disimpan
        Server->>Email: Kirim Konfirmasi Booking + Lampiran file .ics
        Server-->>UI: 201 Created (appointmentId, details)
        UI-->>Client: Tampilkan Layar Sukses Janji Temu
        Email-->>Client: Terima Email Konfirmasi & Undangan Kalender
    end

    Note over Client,Concierge: Hari Kunjungan Butik
    Concierge->>DB: Buka Panel Admin /admin/appointments (Lihat Agenda Hari Ini)
    Concierge->>Concierge: Siapkan Karya Perhiasan di Ruang Privat
    Client->>Concierge: Tamu Tiba di Butik untuk Sesi Privat
    Concierge->>Server: Update Janji Temu ke Status "completed"
```

---

## 2. Tabel Langkah Pengguna & Sistem

| Langkah | Aksi Pengguna (User Action) | Respon Sistem (System Action) | State / Data Terkait |
|:---:|---|---|---|
| **1** | Membuka rute `/appointments` atau mengklik "Book Boutique Appointment" pada halaman produk. | Sistem menampilkan antarmuka pemesanan janji temu. Jika berasal dari halaman produk, karya perhiasan otomatis terpasang sebagai referensi. | State `pieceId` tersimpan. |
| **2** | Memilih lokasi butik fisik (misal: Bifrost Flagship Boutique). | Sistem menetapkan `boutiqueId` aktif dan memuat informasi alamat serta jam operasional butik. | Butik terpilih. |
| **3** | Memilih tanggal kunjungan pada kalender (rentang H+1 hingga H+30). | Sistem melakukan kueri asinkron ke `GET /api/appointments/slots`, menghitung slot 60 menit yang belum terisi, dan merender tombol slot jam. | Daftar slot jam ter-render. |
| **4** | Memilih jam kunjungan yang tersedia (misal: pukul 15:00 - 16:00). | Sistem mengunci pilihan waktu pada antarmuka dan membuka formulir informasi kontak tamu. | State waktu kunjungan terkunci. |
| **5** | Mengisi data diri (nama lengkap, email, nomor telepon) dan catatan khusus (misal: preferensi potongan berlian). | Sistem memvalidasi kelengkapan format email dan nomor telepon secara real-time via Zod. | Form tervalidasi. |
| **6** | Menekan tombol "Confirm Private Appointment". | Sistem mengeksekusi `POST /api/appointments`. Server memverifikasi ketiadaan konflik slot secara atomik di database, menyimpan rekaman dengan status `confirmed`, dan memicu pembuatan berkas undangan kalender (.ics). | Record `appointments` tersimpan. |
| **7** | Menerima layar konfirmasi sukses dan email undangan kalender. | Email konfirmasi tiba di inbox tamu lengkap dengan tautan Google Maps butik dan berkas kalender yang dapat langsung ditambahkan ke perangkat seluler. | Email terkirim via Resend. |
| **8** | Staf concierge butik menyambut kedatangan tamu pada hari yang ditentukan. | Staf butik membuka `/admin/appointments`, mempersiapkan karya perhiasan di ruang pamer pribadi, dan memperbarui status janji temu menjadi `completed`. | Status janji temu: `completed`. |

---

## 3. Penanganan Error & Kasus Tepi (Edge Cases)

| Skenario Kasus Tepi | Gejala / Kondisi | Solusi & Penanganan Sistem |
|---|---|---|
| **Tabrakan Slot Waktu (Race Condition / 409 Conflict)** | Dua calon tamu memilih dan menekan tombol konfirmasi pada slot jam yang sama secara bersamaan. | Transaksi basis data menerapkan pengecekan ketersediaan unik. Pengguna pertama berhasil, pengguna kedua menerima respon `409 Conflict` dengan pesan santun dan pembaruan slot jam secara otomatis. |
| **Upaya Pemesanan Hari Ini (Same-day Booking)** | Pengguna mencoba memesan jadwal kunjungan pada hari yang sama. | Kalender secara default menonaktifkan (*disabled*) tanggal hari ini; pemesanan hanya diizinkan minimal H+1 untuk memberikan waktu persiapan staf concierge. |
| **Pembatalan Janji Temu oleh Klien** | Tamu tidak dapat hadir dan mengklik tautan pembatalan pada email konfirmasi. | Sistem memverifikasi token pembatalan, mengubah status janji temu menjadi `cancelled`, membuka kembali slot jam tersebut untuk publik, dan memberitahukan staf butik. |
| **Tamu Tidak Hadir Tanpa Pemberitahuan (No-Show)** | Waktu janji temu telah berlalu lebih dari 30 menit namun tamu tidak hadir. | Staf admin menandai status janji temu sebagai `no_show` di panel admin untuk keperluan catatan historis dan evaluasi kapasitas butik. |
| **Spam / Bot Menyerang Form Janji Temu** | Skrip otomatis mengirimkan ratusan permintaan booking per menit. | Sistem menerapkan pembatasan rate limit 10 request per menit per IP, validasi honeypot field, dan verifikasi format email standar RFC 5322. |
