# 🔍 FLOW — Alur Penelusuran & Eksplorasi Karya (BROWSING_FLOW)

> **Aktor:** Pengunjung Publik, Pembeli Tamu, Pelanggan Terdaftar  
> **Cakupan:** Halaman Beranda (`/`) → Katalog (`/collections`) / Pencarian Berlian (`/diamonds`) → Halaman Detail Produk (`/pieces/[slug]`) → Jalur Hibrida (Keranjang / Konsultasi / Janji Temu).

---

## 1. Diagram Alur (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor User as Pengunjung
    participant Client as Peramban (Next.js Client)
    participant Server as Server (Next.js RSC / API)
    participant DB as PostgreSQL (Prisma)

    User->>Client: Buka Halaman Beranda (/)
    Client->>Server: Request Halaman Beranda (RSC)
    Server->>DB: Query Featured Pieces & Collections
    DB--Server: Data Kurasi Unggulan
    Server-->>Client: Render HTML Statis / ISR

    alt Jalur Penelusuran Koleksi
        User->>Client: Klik Menu "Collections" atau Kategori Khusus
        Client->>Server: Navigasi ke /collections?category=rings
        Server->>DB: Query Pieces dengan Filter & Paging
        DB--Server: Daftar Produk Terbit (is_published=true)
        Server-->>Client: Render Grid Kartu Perhiasan
    else Jalur Pencarian Berlian 4Cs
        User->>Client: Buka "Diamond Search" (/diamonds)
        User->>Client: Pilih Origin (Natural/Lab-Grown) & Geser Slider Karat
        Client->>Server: Fetch /api/pieces?origin=natural&carat_min=1.0
        Server->>DB: Query Diamonds dengan Kriteria 4Cs
        DB--Server: Hasil Berlian Bersertifikat
        Server-->>Client: Render Tabel & Kartu Hasil Berlian
    end

    User->>Client: Klik Salah Satu Karya Perhiasan
    Client->>Server: Navigasi ke /pieces/[slug]
    Server->>DB: Query Detail Piece + Diamonds + Images + Variants
    DB--Server: Data Lengkap Karya Perhiasan
    Server-->>Client: Render Halaman Detail Produk (PDP)

    alt Pilihan Aksi Hibrida (Hybrid Path)
        User->>Client: Klik "Add to Bag" (Pembelian Langsung)
        Client->>Client: Buka Drawer Keranjang dengan Varian Terpilih
    else Konsultasi Kustom
        User->>Client: Klik "Request Consultation"
        Client->>Client: Buka Modal / Form Konsultasi dengan PieceID Terlampir
    else Janji Temu Butik
        User->>Client: Klik "Book Boutique Viewing"
        Client->>Client: Arahkan ke Form Janji Temu dengan PieceID Terlampir
    end
```

---

## 2. Tabel Langkah Pengguna & Sistem

| Langkah | Aksi Pengguna (User Action) | Respon Sistem (System Action) | State / Data Terkait |
|:---:|---|---|---|
| **1** | Mengakses alamat situs Bifrost Jewellery (`/`). | Server menyajikan halaman beranda hasil ISR dengan hero fotografi tenang, kurasi mahakarya unggulan, dan akses navigasi utama. | Status HTTP 200, cache `s-maxage=3600`. |
| **2** | Memilih kategori perhiasan (misal: Cincin Berlian) pada navbar. | Peramban melakukan prefetch dan menampilkan `/collections?category=rings`. Server mengambil karya dengan `is_published = true`. | Parameter URL `?category=rings` aktif. |
| **3** | Menerapkan filter lanjutan (material logam platinum, harga, atau sorting). | Client melakukan update shallow route tanpa refresh penuh. Query dieksekusi efisien menggunakan indeks basis data. | URL merefleksikan query string lengkap. |
| **4** | Atau: Mengakses `/diamonds` dan menentukan parameter 4Cs (Natural, Round, 1.0 - 2.0 ct). | Client mendebounce input slider (300ms) dan memperbarui daftar berlian bersertifikat GIA/IGI yang cocok. | State filter gemologi sinkron dengan URL. |
| **5** | Mengklik salah satu karya perhiasan yang menarik perhatian. | Server merender `/pieces/[slug]`, memuat galeri multi-sudut, tabel spesifikasi gemologi 4Cs, nomor sertifikat, dan selektor ukuran cincin. | Validasi data slug di DB. Jika tidak ditemukan → render `not-found.tsx`. |
| **6** | Memilih ukuran cincin (*ring size*) yang sesuai. | Client mengaktifkan tombol aksi utama "Add to Bag" dan mengaitkan `variantId` yang tepat. | State varian aktif di antarmuka pengguna. |
| **7A** | Memilih opsi "Add to Bag". | Client menambahkan varian ke keranjang (cookie sesi/DB) dan memunculkan *Cart Drawer* ringkas. | Item tersimpan di `cart_lines`. |
| **7B** | Memilih opsi "Request Consultation". | Client menampilkan formulir konsultasi privat dengan foto dan nomor SKU perhiasan tersemat otomatis. | Komponen modal membawa referensi `pieceId`. |
| **7C** | Memilih opsi "Book Boutique Viewing". | Client mengarahkan ke alur `/appointments` dengan karya terpilih yang akan disiapkan staf butik. | Parameter query `?pieceId={id}` diteruskan. |

---

## 3. Penanganan Error & Kasus Tepi (Edge Cases)

| Skenario Kasus Tepi | Gejala / Kondisi | Solusi & Penanganan Sistem |
|---|---|---|
| **Karya Ditarik / Tidak Ditemukan** | Pengguna mengakses URL `/pieces/[slug]` untuk karya yang sudah di-unpublish atau tidak ada. | Server mengembalikan respon `404 Not Found` dan menampilkan komponen `not-found.tsx` berdesain editorial elegan dengan tombol kembali ke katalog. |
| **Filter Menghasilkan 0 Hasil** | Kombinasi filter 4Cs terlalu spesifik (misal: Berlian Natural, Marquise, Flawless, >5 ct di bawah Rp 50 juta). | Menampilkan `SearchEmptyState` yang sopan dengan saran pelonggaran kriteria dan tombol "Reset Filters" atau tautan ke layanan *Bespoke Sourcing*. |
| **Karya Berstatus Reserved / Sold Out** | Pengguna membuka karya yang baru saja dipesan oleh pelanggan lain. | Tombol "Add to Bag" dinonaktifkan otomatis dan digantikan oleh pesan informatif "Currently Reserved" serta tombol sekunder "Inquire on Similar Piece". |
| **Upaya Pembelian Tanpa Memilih Ukuran** | Pengguna menekan tombol beli pada cincin tanpa memilih ukuran jari. | Antarmuka memberikan feedback visual halus pada selektor ukuran cincin dan menyorot tautan "Size Guide" tanpa alert browser yang mengganggu. |
| **Koneksi Internet Lambat** | Pengguna membuka galeri foto beresolusi tinggi di jaringan lemah. | Foto studio menggunakan placeholder blur berbobot rendah (*blur-up WebP LQIP*) dan memuat gambar ukuran penuh secara progresif. |
