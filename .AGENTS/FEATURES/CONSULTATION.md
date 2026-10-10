# 💌 FEATURE — Formulir Konsultasi Privat & Kustom (CONSULTATION)

> **Status:** Planned  
> **Scope:** Bespoke Sourcing & Private Inquiries (Hybrid Commerce Path)  
> **Primary Goal:** Memfasilitasi jalur transaksi konsultatif (*hybrid path*) bagi calon pembeli yang menginginkan karya pesanan khusus (*bespoke commission*), pencarian berlian langka (*diamond sourcing*), atau pertanyaan mendalam mengenai karya perhiasan tertentu tanpa melalui keranjang belanja langsung.

---

## 1. Ringkasan (Overview)

Tidak semua mahakarya perhiasan bernilai ratusan juta rupiah dibeli melalui satu kali klik tombol keranjang belanja. Banyak klien menginginkan dialog personal dengan tim kurator dan ahli gemologi Bifrost Jewellery untuk menyesuaikan desain cincin, memilih batu berlian dengan sertifikasi spesifik, atau merancang perhiasan keluarga (*family heirloom*).

Fitur Konsultasi Privat menyediakan saluran penghubung berstandar *concierge*. Klien dapat mengirimkan formulir konsultasi terarah dari halaman karya perhiasan tertentu (dengan referensi produk terlampir otomatis) maupun formulir pesanan khusus mandiri (*bespoke inquiry form*). Setiap pesan yang masuk diproses ke dalam antrean *inquiries* panel admin dan secara instan diteruskan ke email tim concierge butik.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Calon Pengantin:** Saya ingin mengonsultasikan pembuatan cincin tunangan kustom (*bespoke ring*) dengan membagikan preferensi batu permata, anggaran, dan inspirasi desain saya kepada perancang Bifrost.
- **Sebagai Pengagum Karya Eksklusif:** Saya ingin mengajukan pertanyaan mengenai karya berstatus *reserved* atau *made_to_order* dan meminta penawaran waktu pengerjaan khusus.
- **Sebagai Klien Internasional:** Saya ingin meminta opsi metode pengiriman khusus atau pengaturan pembayaran privat di luar gerbang pembayaran reguler.
- **Sebagai Tim Concierge Butik:** Saya ingin menerima notifikasi instan saat ada permintaan konsultasi baru, melihat riwayat pesan klien, dan memperbarui status penanganan (*new* → *contacted* → *closed*).

---

## 3. Aturan Bisnis (Business Rules)

1. **Fleksibilitas Jalur Hibrida (Hybrid Path Integration):**
   - Jika formulir konsultasi dibuka dari tombol "Request Consultation" pada halaman produk, sistem secara otomatis mengaitkan `piece_id` karya tersebut ke dalam data inquiry.
   - Jika dibuka dari halaman umum `/consultation`, bidang referensi karya bersifat opsional (`piece_id = null`).
2. **Validasi Kontak Klien:**
   - Nama lengkap, alamat email aktif, dan pesan/kebutuhan wajib diisi.
   - Nomor telepon bersifat opsional namun sangat direkomendasikan untuk komunikasi concierge via WhatsApp atau panggilan privat.
3. **Siklus Status Inquiry:**
   - `new`: Pertanyaan baru masuk dan belum ditindaklanjuti oleh tim concierge.
   - `contacted`: Staf telah menghubungi klien melalui email atau telepon.
   - `closed`: Konsultasi selesai (berhasil beralih menjadi pesanan kustom atau diarsipkan).
4. **Pencegahan Spam & Rate Limiting:**
   - Endpoint formulir konsultasi dibatasi maksimal 5 pengiriman per menit per alamat IP.
   - Dilengkapi *honeypot hidden field* untuk menangkal bot spam otomatis tanpa mengganggu kenyamanan pengguna manusia dengan captcha visual yang merusak estetika quiet luxury.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini berpusat pada tabel:

- `inquiries`: Data rekaman formulir konsultasi (`id`, `user_id`, `name`, `email`, `phone`, `piece_id`, `message`, `status`, `notes`, `created_at`, `updated_at`).
  - Enum `status`: `new`, `contacted`, `closed`.
- `pieces`: Relasi karya perhiasan yang diminati (jika diajukan dari halaman detail produk).
- `users`: Relasi akun jika pengirim adalah pengguna yang sedang login.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### 1. Endpoint Pengiriman Konsultasi

- **`POST /api/inquiries`**
  - **Headers:** `Content-Type: application/json`
  - **Request Body (Zod Validated):**
    ```json
    {
      "name": "Lady Catherine",
      "email": "catherine@example.com",
      "phone": "+6281199887766",
      "pieceId": "c1a2b3c4-d5e6-7f8a-9b0c-1d2e3f4a5b6c",
      "message": "Saya sangat tertarik dengan desain Celestial Solitaire ini, namun ingin menanyakan kemungkinan menggunakan batu berlian Oval Cut 2.0ct dengan spesifikasi warna D dan kejernihan VVS1.",
      "honeypot": ""
    }
    ```
  - **Respon 201 Created:**
    ```json
    {
      "success": true,
      "message": "Terima kasih telah menghubungi Bifrost Jewellery. Concierge kami akan menghubungi Anda secara privat dalam kurun waktu 1x24 jam kerja."
    }
    ```
  - **Respon 400 Bad Request:**
    ```json
    {
      "success": false,
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Format email tidak valid atau pesan konsultasi terlalu pendek."
      }
    }
    ```

### 2. Server Action Manajemen Inquiry Admin

- **`updateInquiryStatusAction({ inquiryId: string, status: 'new' | 'contacted' | 'closed', internalNotes?: string })`**
  - Khusus untuk pengguna berotorisasi `admin` dan `super_admin`.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `ConsultationForm`: Formulir masukan elegan dengan bidang bergaris bawah halus atau kotak border hairline `--ql-line`, transisi fokus warna `--ql-gold-deep`, dan tombol submit tenang "Send Consultation Request".
- `AttachedPieceBadge`: Kotak pratinjau kecil di bagian atas formulir yang memperlihatkan thumbnail foto, nama karya, dan nomor SKU karya perhiasan yang sedang ditanyakan.
- `BespokeInspirationUploader`: Opsi unggahan gambar sketsa atau inspirasi perhiasan (JPG/PNG maks. 5MB) untuk pesanan kustom murni.
- `ConciergeAssuranceNote`: Catatan etika privasi tenang yang menegaskan bahwa seluruh percakapan dan data klien dijaga kerahasiaannya secara ketat (*strict confidentiality*).

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Formulir konsultasi yang diajukan dari halaman produk secara otomatis menyertakan referensi `piece_id` yang benar.
- [ ] Pengiriman formulir yang valid menyimpan data baru ke tabel `inquiries` dengan status default `new`.
- [ ] Sistem memicu email notifikasi instan ke kotak masuk tim concierge (`<TBD: concierge email>`) via Resend.
- [ ] Email konfirmasi tanda terima otomatis terkirim ke klien yang mengajukan pertanyaan.
- [ ] Bidang honeypot yang terisi oleh bot secara otomatis menggagalkan pengiriman tanpa memunculkan pesan error teknis.
- [ ] Di panel admin, staf dapat melihat detail inquiry, nomor kontak, pesan, dan memperbarui status penanganan menjadi `contacted` atau `closed`.

---

## 8. Dokumen Terkait

- [PRODUCT_DETAIL.md](./PRODUCT_DETAIL.md) — Titik masuk tombol "Request Consultation"
- [ADMIN.md](./ADMIN.md) — Manajemen antrean inquiries di panel internal
- [UI_UX.md](../UI_UX.md) — Panduan tata letak formulir dan aturan penulisan (*tone of voice*)
