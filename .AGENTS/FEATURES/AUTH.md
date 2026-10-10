# 🔐 FEATURE — Otentikasi & Kontrol Akses RBAC (AUTH)

> **Status:** Planned  
> **Scope:** Authentication & Authorization (Auth.js v5 + RBAC Security)  
> **Primary Goal:** Mengamankan akses platform Bifrost Jewellery menggunakan Auth.js v5 (NextAuth.js), menerapkan matriks kontrol akses berbasis peran (Role-Based Access Control / RBAC) yang ketat, dan melindungi rute admin melalui verifikasi sesi di sisi server (*server-side session check*).

---

## 1. Ringkasan (Overview)

Modul Otentikasi & RBAC menjamin keamanan data pelanggan dan integritas operasional platform Bifrost Jewellery. Arsitektur otentikasi dibangun di atas **Auth.js v5** dengan strategi JWT/Database Session terenkripsi, memfasilitasi login pelanggan menggunakan email dan kata sandi yang di-hash dengan algoritma Argon2/Bcrypt, serta alur undangan khusus (*invitation flow*) bagi staf admin internal.

Sistem menerapkan prinsip pertahanan berlapis (*defense-in-depth*): selain pemeriksaan awal pada `middleware.ts`, setiap React Server Component dan Server Action internal wajib memverifikasi klaim peran (*role claim*) langsung dari sesi server sebelum menjalankan operasi data apa pun.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Pelanggan:** Saya ingin mendaftar dan masuk ke akun saya menggunakan email dan kata sandi yang aman, serta tetap dalam kondisi login melalui cookie sesi yang terlindungi.
- **Sebagai Pelanggan yang Lupa Kata Sandi:** Saya ingin mereset kata sandi saya dengan aman melalui tautan token berbatas waktu yang dikirim ke email saya.
- **Sebagai Staf Admin:** Saya ingin masuk ke panel admin `/admin` menggunakan kredensial resmi dan mengakses fitur pengelolaan katalog, pesanan, dan janji temu sesuai izin peran saya.
- **Sebagai Super Admin:** Saya ingin mengelola akun admin lain, mengonfigurasi pengaturan sistem, dan menyetujui mutasi keuangan tingkat tinggi.

---

## 3. Aturan Bisnis & Matriks RBAC (Business Rules)

### 1. Definisi Peran (Roles Definition)

Platform mendefinisikan 4 hierarki peran:
- `super_admin`: Pemilik sistem / manajer eksekutif butik dengan akses penuh tanpa batas.
- `admin`: Staf operasional, manajer katalog, dan concierge butik.
- `customer`: Pelanggan terdaftar yang berbelanja atau memiliki akun.
- `guest`: Pengunjung publik tanpa akun (akses publik, checkout tamu, pelacakan pesanan via nomor resi & email).

### 2. Matriks Izin Akses (RBAC Permission Matrix)

| Modul / Kemampuan | `guest` | `customer` | `admin` | `super_admin` |
|---|:---:|:---:|:---:|:---:|
| Jelajah Katalog & Detail Produk | ✅ | ✅ | ✅ | ✅ |
| Checkout & Pelacakan Pesanan Sendiri | ✅ | ✅ | ✅ | ✅ |
| Buku Alamat & Wishlist Akun | ❌ | ✅ | ✅ | ✅ |
| Booking Janji Temu & Kirim Konsultasi | ✅ | ✅ | ✅ | ✅ |
| Akses Dashboard Admin (`/admin`) | ❌ | ❌ | ✅ | ✅ |
| Kelola Katalog (Pieces & Diamonds) | ❌ | ❌ | ✅ | ✅ |
| Pemenuhan Pesanan (`processing`, `shipped`) | ❌ | ❌ | ✅ | ✅ |
| Verifikasi Pembayaran Transfer Manual | ❌ | ❌ | ✅ | ✅ |
| Kelola Janji Temu & Inquiries | ❌ | ❌ | ✅ | ✅ |
| Otorisasi Pengembalian Dana (`refunded`) | ❌ | ❌ | ❌ | ✅ |
| Kelola Akun Admin & Konfigurasi Sistem | ❌ | ❌ | ❌ | ✅ |

### 3. Proteksi Server-Side Mutlak:
- Pengecekan sesi pada `middleware.ts` hanya berfungsi sebagai penyaring rute awal (*routing guard*).
- **Aturan Wajib:** Setiap Server Action admin wajib memanggil fungsi utilitas pembuktian sesi:
  ```typescript
  const session = await auth();
  if (!session?.user || !['admin', 'super_admin'].includes(session.user.role)) {
    throw new UnauthorizedError('Akses ditolak: Memerlukan hak akses administratif.');
  }
  ```

---

## 4. Integrasi Data (DATABASE.md)

Modul ini berpusat pada tabel:

- `users`: Entitas pengguna sistem (`id`, `email`, `password_hash`, `full_name`, `phone`, `role`, `is_active`, `email_verified_at`, `created_at`, `updated_at`).
  - Enum `role`: `super_admin`, `admin`, `customer`.
- `sessions` / `accounts`: Tabel terkelola Auth.js untuk pelacakan token sesi login aktif.
- `verification_tokens`: Token untuk verifikasi email dan reset kata sandi akun.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### 1. Handler Rute Auth.js

- **`POST /api/auth/[...nextauth]`** — Endpoint internal bawaan Auth.js v5 untuk login, logout, dan manajemen sesi.

### 2. Server Action Otentikasi

- **`loginAction(credentials: LoginSchema)`**
  - Menerima email dan password, memverifikasi hash via Bcrypt/Argon2, dan menerbitkan cookie sesi `authjs.session-token` bertanda tangan `HttpOnly`, `SameSite=Lax`, `Secure`.
- **`registerAction(data: RegisterSchema)`**
  - Mendaftarkan akun pelanggan baru dengan peran default `customer`.
- **`requestPasswordResetAction(email: string)`**
  - Menerbitkan token reset kata sandi berdurasi kedaluwarsa 60 menit dan mengirimkannya ke email pelanggan via Resend.
- **`resetPasswordAction(token: string, newPassword: string)`**
  - Memverifikasi token dan memperbarui hash kata sandi pengguna.

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `LoginForm`: Formulir login minimalis dengan label bersih, input kata sandi dengan toggle intip visibilitas tenang, dan tautan "Forgot password?".
- `RegisterForm`: Formulir pendaftaran akun pelanggan dengan indikator kekuatan kata sandi yang elegan (minimal 8 karakter, huruf besar, dan angka).
- `PasswordResetModal`: Modal pengajuan reset kata sandi dengan feedback email terkirim yang informatif.
- `AdminUnauthorizedView`: Halaman penolakan akses formal jika pengguna non-admin mencoba membuka rute `/admin`.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Pengguna dengan kredensial yang valid berhasil login dan menerima cookie sesi berbendera `HttpOnly` dan `Secure`.
- [ ] Pengguna dengan peran `customer` ditolak saat mengakses rute `/admin/*` dan dialihkan ke `/login?error=Unauthorized`.
- [ ] Pengguna tanpa sesi diarahkan ke halaman login saat mencoba mengakses `/account` atau `/admin`.
- [ ] Password disimpan dalam bentuk hash terenkripsi aman; tidak ada plaintext password yang tersimpan di basis data atau log sistem.
- [ ] Mutasi Server Action administratif menolak eksekusi jika sesi tidak memiliki peran `admin` atau `super_admin` dengan melempar error otorisasi.
- [ ] Rate limiting login aktif pada batas 5 percobaan login gagal per menit per alamat IP untuk mencegah serangan *brute force*.

---

## 8. Dokumen Terkait

- [ADMIN.md](./ADMIN.md) — Panel administratif yang dilindungi oleh modul ini
- [ACCOUNT.md](./ACCOUNT.md) — Pengalaman akun pengguna terotentikasi
- [SERVER_ACTION_PATTERN.md](../PATTERNS/SERVER_ACTION_PATTERN.md) — Pola otorisasi sesi di Server Actions
