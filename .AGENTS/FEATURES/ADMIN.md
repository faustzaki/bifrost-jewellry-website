# 🛡️ FEATURE — Panel Admin Internal (ADMIN)

> **Status:** Planned  
> **Scope:** Internal In-App Backoffice (`/admin`)  
> **Primary Goal:** Menyediakan pusat kendali operasional terintegrasi dalam aplikasi Next.js untuk mengelola inventaris mahakarya perhiasan dan berlian, memproses dan menyetujui pembayaran manual, mengelola siklus pesanan, mengoordinasikan janji temu butik, menindaklanjuti pesan konsultasi, dan mengawasi pengguna sistem.

---

## 1. Ringkasan (Overview)

Sesuai keputusan arsitektur (ADR-007), Bifrost Jewellery tidak bergantung pada CMS pihak ketiga yang terpisah. Seluruh kebutuhan pengelolaan butik digital dijalankan melalui panel admin internal terintegrasi pada rute `/admin`.

Panel admin dibangun menggunakan React Server Components untuk pengambilan data efisien dan Server Actions bervalidasi Zod untuk seluruh operasi mutasi data. Antarmuka panel didesain dengan tata letak dasbor profesional yang bersih, tabel data berkecepatan tinggi dengan penyaringan instan, formulir input spesifikasi gemologi berlian berstandar GIA/IGI, serta sistem audit pencatatan aksi staf.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Kurator Perhiasan (Admin):** Saya ingin menambah, mengedit, dan mengarsipkan karya perhiasan baru, mengunggah foto studio resolusi tinggi ke Vercel Blob, serta mengisi spesifikasi 4Cs berlian bersertifikat.
- **Sebagai Staf Keuangan (Admin):** Saya ingin melihat daftar transaksi pembayaran transfer bank manual, memverifikasi kecocokan bukti transfer dengan mutasi bank, dan menekan tombol persetujuan (*approve*) agar pesanan segera diproses.
- **Sebagai Manajer Pemenuhan (Admin/Ops):** Saya ingin memperbarui status pesanan dari `paid` menjadi `processing` dan `shipped`, serta memasukkan nomor resi kurir berasuransi khusus.
- **Sebagai Concierge Butik (Admin):** Saya ingin melihat kalender janji temu harian, membaca preferensi tamu, menandai kehadiran tamu (*completed*), dan memperbarui status tiket konsultasi (*inquiries*).
- **Sebagai Super Admin:** Saya ingin melihat ringkasan metrik penjualan butik, mengundang akun admin baru, dan mengubah peran staf.

---

## 3. Aturan Bisnis (Business Rules)

1. **Struktur Modul Panel Admin:**
   Panel admin terdiri dari 6 modul kerja:
   - **Dashboard Ringkasan:** Metrik pendapatan kotor, pesanan pending pembayaran, janji temu mendatang hari ini, dan inquiry baru.
   - **Katalog & Berlian (`/admin/pieces`):** CRUD karya perhiasan, manajemen foto (reorder/upload), spesifikasi berlian 4Cs, dan pengaturan status penerbitan (`is_published`).
   - **Pesanan & Keuangan (`/admin/orders`):** Daftar pesanan, verifikasi transfer manual, pengubahan status pemenuhan, dan input resi kurir.
   - **Janji Temu Butik (`/admin/appointments`):** Kalender reservasi kunjungan butik, catatan khusus tamu, dan pemenuhan sesi.
   - **Konsultasi & Concierge (`/admin/inquiries`):** Kotak masuk konsultasi pesanan khusus dan pencarian berlian langka.
   - **Pengguna & Staf (`/admin/users`):** Pengelolaan staf internal (khusus `super_admin`).
2. **Integritas Mutasi Data:**
   - Semua mutasi data (Create, Update, Delete) di panel admin **WAJIB** menggunakan Server Actions yang terisolasi dengan validasi Zod dan pengecekan otorisasi sesi server.
   - Dilarang membuat endpoint REST terbuka untuk mutasi admin internal.
3. **Penyimpanan Berkas Foto & Sertifikat:**
   - Unggahan foto perhiasan dan dokumen sertifikat diunggah langsung ke Vercel Blob Storage melalui token aman sisi server (`BLOB_READ_WRITE_TOKEN`).
   - Berkas dioptimasi ke format WebP otomatis sebelum disimpan permanen.
4. **Audit Trail Verifikasi:**
   - Saat admin menyetujui pembayaran transfer manual, sistem secara permanen merekam `verified_by = session.user.id` dan `verified_at = new Date()`.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini memiliki akses penuh ke seluruh entitas basis data:

- `pieces`, `piece_images`, `piece_variants`, `diamonds`, `categories`, `collections`
- `orders`, `order_lines`, `payments`
- `appointments`, `boutiques`, `inquiries`
- `users`

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

Seluruh operasi administratif menggunakan **Server Actions** di `src/app/admin/**/actions.ts`:

### 1. Manajemen Katalog
- **`createPieceAction(data: CreatePieceSchema)`**
- **`updatePieceAction(id: string, data: UpdatePieceSchema)`**
- **`deletePieceAction(id: string)`**
- **`attachDiamondAction(pieceId: string, diamondData: DiamondSchema)`**

### 2. Manajemen Pesanan & Pembayaran
- **`approveManualPaymentAction(paymentId: string)`**
- **`rejectManualPaymentAction(paymentId: string, reason: string)`**
- **`updateOrderFulfillmentAction({ orderId: string, status: OrderStatus, trackingNumber?: string, courierName?: string })`**

### 3. Manajemen Janji Temu & Inquiries
- **`updateAppointmentStatusAction({ appointmentId: string, status: AppointmentStatus })`**
- **`updateInquiryStatusAction({ inquiryId: string, status: 'new' | 'contacted' | 'closed' })`**

### 4. Manajemen Pengguna
- **`inviteAdminUserAction({ email: string, fullName: string, role: 'admin' })`**

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `AdminShell`: Tata letak induk admin dengan sidebar navigasi vertikal, profil admin aktif, tombol logout cepat, dan area konten responsif.
- `AdminDataTable`: Tabel data dinamis dengan fitur pencarian teks, filter status, pengurutan kolom, dan pagination server-side.
- `DiamondSpecificationForm`: Formulir masukan data berlian dengan selektor origin (Natural vs Lab-Grown), dropdown bentuk batu, input desimal karat berpresisi, dan input nomor sertifikat GIA/IGI.
- `PaymentVerificationCard`: Panel peninjauan bukti transfer manual yang menampilkan gambar bukti bayar berdampingan dengan jumlah nominal pesanan serta tombol tindakan "Approve Payment" dan "Reject".
- `StatusSelectDropdown`: Dropdown pengubah status pesanan atau janji temu dengan konfirmasi modal pengaman untuk tindakan kritis.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Seluruh rute di bawah `/admin` dibentengi oleh proteksi sesi server; pengguna tanpa peran `admin` atau `super_admin` otomatis ditolak.
- [ ] Menambah karya perhiasan baru berhasil menyimpan data relasi `pieces`, `diamonds`, dan `piece_variants` dalam satu transaksi Prisma atomik.
- [ ] Menyetujui bukti transfer manual berhasil mengubah status pembayaran dan pesanan menjadi `paid`, serta mencatat ID admin penanggung jawab.
- [ ] Memperbarui status pesanan menjadi `shipped` mewajibkan pengisian nomor resi kurir dan nama kurir.
- [ ] Setiap Server Action admin memanggil `revalidatePath` pada halaman publik terkait agar cache ISR diperbarui secara instan.
- [ ] Akses ke modul pengelolaan staf internal (`/admin/users`) hanya dapat dibuka oleh pengguna dengan peran `super_admin`.

---

## 8. Dokumen Terkait

- [AUTH.md](./AUTH.md) — Matriks otorisasi RBAC dan proteksi sesi
- [PAYMENT.md](./PAYMENT.md) — Alur verifikasi pembayaran manual
- [SERVER_ACTION_PATTERN.md](../PATTERNS/SERVER_ACTION_PATTERN.md) — Pola Server Action aman dengan validasi Zod
