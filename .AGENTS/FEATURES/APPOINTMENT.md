# 🏛️ FEATURE — Janji Temu Butik (APPOINTMENT)

> **Status:** Planned  
> **Scope:** Boutique Visits & Online Scheduling (Public Booking + Admin Concierge)  
> **Primary Goal:** Memberikan kemudahan bagi calon pembeli dan kolektor perhiasan untuk memesan jadwal kunjungan privat ke butik fisik Bifrost Jewellery, memilih karya perhiasan yang ingin dilihat secara langsung, memvalidasi ketersediaan slot waktu tanpa bentrok (*conflict detection*), serta memfasilitasi persiapan dan penerimaan tamu oleh staf butik.

---

## 1. Ringkasan (Overview)

Perhiasan kelas tinggi (*high jewellery*) sering kali membutuhkan pengalaman taktil langsung: merasakan bobot platinum, mengamati kilau dispersi cahaya berlian di bawah pencahayaan alami, atau berdiskusi mendalam dengan gemologis berlisensi. Modul Janji Temu Butik menjembatani kehadiran butik daring dengan butik fisik Bifrost Jewellery.

Calon klien dapat memilih butik fisik yang ingin dikunjungi, memilih tanggal dan slot waktu konsultasi privat yang tersedia, serta menentukan karya perhiasan atau jenis berlian spesifik yang ingin dipersiapkan oleh staf butik sebelum kedatangan mereka. Sistem menjamin tidak terjadi benturan jadwal (*overbooking*) melalui pengecekan konflik slot waktu di server, dan secara otomatis mengirimkan email konfirmasi kalender (iCalendar / `.ics`).

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Calon Klien:** Saya ingin memilih tanggal dan jam kunjungan ke butik yang masih kosong, agar saya mendapatkan sesi konsultasi privat yang tenang dan eksklusif tanpa harus mengantre.
- **Sebagai Pengagum Karya Tertentu:** Saya ingin menyertakan karya perhiasan tertentu (dari halaman produk) saat memesan janji temu, agar staf butik dapat mempersiapkan karya tersebut di ruang pajang privat sebelum saya tiba.
- **Sebagai Tamu Butik:** Saya ingin menerima email konfirmasi instan beserta lampiran undangan kalender (.ics) dan peta petunjuk arah butik, agar saya dapat menyimpannya di kalender ponsel saya.
- **Sebagai Staf Butik / Concierge:** Saya ingin melihat daftar seluruh janji temu harian di panel admin, mengetahui preferensi tamu serta karya yang diminati, dan menandai status kedatangan tamu (*completed* atau *no-show*).

---

## 3. Aturan Bisnis (Business Rules)

1. **Durasi Slot & Batas Kapasitas:**
   - Setiap sesi janji temu privat berdurasi 60 menit.
   - Setiap slot waktu hanya dapat dipesan oleh maksimal 1 tamu/rombongan privat per staf butik yang bertugas (kapasitas default: 1 pemesanan per slot per butik).
   - Pemesanan dapat dilakukan paling cepat 24 jam sebelum waktu kunjungan (mencegah pemesanan mendadak yang tidak siap dilayani).
   - Rentang pemesanan maksimal adalah 30 hari ke depan.
2. **Pencegahan Konflik Jadwal (Conflict Detection - HTTP 409):**
   - Ketersediaan slot waktu wajib divalidasi ulang saat proses penyimpanan ke basis data dalam transaksi atomik.
   - Jika dua pengguna memesan slot yang sama pada waktu bersamaan, pengguna kedua akan menerima respon penolakan `409 Conflict` dengan pesan meminta memilih slot lain.
3. **Siklus Status Janji Temu (Appointment Lifecycle):**
   - `requested` → Janji temu baru dibuat oleh klien.
   - `confirmed` → Terkonfirmasi otomatis oleh sistem (atau diverifikasi oleh staf concierge).
   - `completed` → Klien telah hadir dan sesi butik selesai dilaksanakan.
   - `cancelled` → Dibatalkan oleh klien atau pihak butik (minimal 6 jam sebelum jadwal).
   - `no_show` → Tamu tidak hadir pada waktu yang dijadwalkan tanpa pemberitahuan.
4. **Data Tamu Fleksibel:**
   - Pemesanan janji temu tidak mewajibkan registrasi akun, namun data tamu (`guest_name`, `guest_email`, `guest_phone`) wajib diisi dan valid.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini berinteraksi dengan:

- `appointments`: Rekaman janji temu (`id`, `user_id`, `guest_name`, `guest_email`, `guest_phone`, `boutique_id`, `scheduled_at`, `status`, `piece_id`, `notes`, `created_at`, `updated_at`).
- `boutiques`: Profil butik fisik (`id`, `name`, `address_line`, `city`, `map_url`, `phone`, `opening_hours`, `is_active`).
- `pieces`: Karya perhiasan yang ingin dilihat saat kunjungan (`piece_id`).
- `users`: Relasi akun pengguna jika tamu telah terdaftar.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### 1. Kueri Slot Ketersediaan Publik

- **`GET /api/appointments/slots`**
  - **Query Parameters:** `boutiqueId={uuid}&date=YYYY-MM-DD`
  - **Respon 200 OK:**
    ```json
    {
      "success": true,
      "data": {
        "boutiqueId": "b1a2b3c4-0000-0000-0000-111122223333",
        "date": "2026-10-15",
        "slots": [
          { "time": "11:00", "available": true },
          { "time": "13:00", "available": false },
          { "time": "15:00", "available": true },
          { "time": "17:00", "available": true },
          { "time": "19:00", "available": false }
        ]
      }
    }
    ```

### 2. Pembuatan Janji Temu

- **`POST /api/appointments`**
  - **Request Body (Zod Validated):**
    ```json
    {
      "boutiqueId": "b1a2b3c4-0000-0000-0000-111122223333",
      "scheduledAt": "2026-10-15T15:00:00+07:00",
      "guestName": "Alexander Hartono",
      "guestEmail": "alexander@example.com",
      "guestPhone": "+6281122334455",
      "pieceId": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
      "notes": "Mencari cincin pertunangan dengan potongan emerald cut."
    }
    ```
  - **Respon 201 Created:**
    ```json
    {
      "success": true,
      "data": {
        "appointmentId": "apt-9988-7766",
        "status": "confirmed",
        "scheduledAt": "2026-10-15T15:00:00+07:00",
        "boutique": {
          "name": "Bifrost Flagship Boutique Jakarta",
          "address": "Pacific Place Mall, Level 1, SCBD, Jakarta Selatan"
        }
      }
    }
    ```
  - **Respon 409 Conflict:**
    ```json
    {
      "success": false,
      "error": {
        "code": "SLOT_UNAVAILABLE",
        "message": "Maaf, slot waktu yang Anda pilih baru saja dipesan oleh tamu lain. Silakan pilih jam atau tanggal alternatif."
      }
    }
    ```

### 3. Server Action Admin

- **`updateAppointmentStatus({ appointmentId: string, status: AppointmentStatus, adminNotes?: string })`**
  - Mengubah status janji temu dan mencatat log tindak lanjut concierge.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `AppointmentDatePicker`: Kalender visual tenang yang menampilkan hari-hari yang dapat dipesan dengan batasan tanggal cerdas (minimal H+1, maksimal H+30).
- `TimeSlotGrid`: Pilihan kisi tombol waktu 60 menit dengan indikator jelas antara slot tersedia dan slot terisi.
- `SelectedPiecePreview`: Kartu ringkas foto dan nama karya perhiasan yang otomatis terlampir jika pemesanan dimulai dari halaman detail produk.
- `BoutiqueLocationSelector`: Tab atau dropdown pilihan lokasi butik fisik (misal: Flagship Jakarta atau Butik Surabaya).
- `ConfirmationModal`: Modal bersuasana tenang yang menampilkan ucapan terima kasih personal, ringkasan waktu kunjungan, dan panduan kedatangan.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Memilih tanggal memuat slot waktu yang benar-benar akurat sesuai pemesanan yang telah ada di basis data.
- [ ] Sistem menolak pemesanan pada slot waktu yang sudah terisi dengan kode status 409 Conflict.
- [ ] Email konfirmasi pemesanan terkirim otomatis ke `guestEmail` melalui layanan Resend dalam waktu kurang dari 10 detik.
- [ ] Lampiran file iCalendar (.ics) berfungsi dan dapat ditambahkan ke Google Calendar / Apple Calendar tanpa error.
- [ ] Staf admin dapat melihat seluruh janji temu terurut berdasarkan waktu kunjungan dan mengubah status pemenuhan menjadi `completed`.
- [ ] Pembatasan rate limit 10 percobaan per menit per alamat IP aktif untuk mencegah eksploitasi spamming form.

---

## 8. Dokumen Terkait

- [BOUTIQUE.md](./BOUTIQUE.md) — Informasi detail lokasi dan jam operasional butik fisik
- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Alur inisiasi janji temu dari halaman produk
- [ADMIN.md](./ADMIN.md) — Pengelolaan daftar janji temu di panel admin
