# GLOSSARY.md — Glosarium Istilah & Definisi Sistem

> **Tujuan:** Dokumen ini menjadi sumber kebenaran tunggal (*single source of truth*) untuk seluruh terminologi bisnis, entitas sistem, status alur kerja, peran pengguna, dan singkatan teknis dalam proyek Bifrost Jewellery. Seluruh pengembang dan AI agent wajib menggunakan penamaan yang konsisten sesuai dokumen ini.

---

## 1. Entitas Inti Sistem (Core Entities)

Setiap entitas memiliki nama baku dalam kode dan database, serta aturan alias terlarang untuk menjaga konsistensi domain.

### 1. Piece
- **Definisi:** Karya perhiasan jadi yang dipajang dan diperdagangkan dalam katalog (misalnya cincin pertunangan, kalung berlian, anting solitaire).
- **Atribut Utama:** `id`, `slug`, `sku`, `name`, `metal`, `price_idr_minor`, `price_usd_minor`, `availability`.
- **_Dilarang digunakan (Do not use):_** `Item`, `Barang`, `Produk Dagangan`, `Product` (dalam teks publik/copywriting gunakan *Piece* atau *Creation*).

### 2. Diamond
- **Definisi:** Batu permata berlian individual yang tersemat pada sebuah Piece atau ditawarkan untuk kustomisasi dudukan (*setting*). Memiliki parameter standar 4Cs dan sertifikat keaslian.
- **Atribut Utama:** `origin`, `carat`, `cut`, `color_grade`, `clarity_grade`, `shape`, `grading_lab`, `certificate_number`.
- **_Dilarang digunakan (Do not use):_** `Batu`, `Permata Biasa`, `Stone` (secara ambigu tanpa konteks berlian).

### 3. 4Cs
- **Definisi:** Standar universal evaluasi kualitas berlian yang ditetapkan secara global: **Carat** (berat/ukuran), **Cut** (kualitas asahan/proporsi), **Color** (derajat ketidakberwarnaan), dan **Clarity** (kejernihan/ketiadaan inklusi internal).
- **Aturan Penulisan:** Wajib ditulis dengan format kapital: **4Cs**.
- **_Dilarang digunakan (Do not use):_** `Empat C`, `4c`, `4-C`.

### 4. Certificate
- **Definisi:** Laporan penilaian mutu dan identitas resmi (*Grading Report*) yang diterbitkan oleh laboratorium gemologi independen berstandar internasional (GIA atau IGI).
- **Atribut Utama:** `grading_lab` (`GIA` | `IGI`), `certificate_number`, `certificate_url`.
- **_Dilarang digunakan (Do not use):_** `Surat Toko`, `Kwitansi Keaslian`, `Nota Jaminan`.

### 5. Setting
- **Definisi:** Rangka kerja logam mulia (cangkang atau dudukan) tempat berlian dipasang dan dikunci dengan aman (misalnya: Prong/Solitaire, Bezel, Pavé, Halo, Channel).
- **Atribut Utama:** `metal` (`gold_18k`, `gold_14k`, `platinum`, `rose_gold_18k`), `setting_type`.
- **_Dilarang digunakan (Do not use):_** `Gagang`, `Ikat Cincin`, `Mount`.

### 6. Collection
- **Definisi:** Kurasi tematik sekumpulan perhiasan (*Pieces*) yang diikat oleh kesatuan narasi desain, era inspirasi, atau gaya visual editorial tertentu.
- **_Dilarang digunakan (Do not use):_** `Kategori Koleksi`, `Grup Produk`.

### 7. Boutique
- **Definisi:** Lokasi butik fisik resmi Bifrost Jewellery tempat pelanggan dapat melihat langsung karya perhiasan, melakukan konsultasi privat, dan mengambil pesanan.
- **_Dilarang digunakan (Do not use):_** `Toko`, `Outlet`, `Cabang Retail`, `Store`.

### 8. Appointment
- **Definisi:** Reservasi sesi kunjungan privat yang dijadwalkan oleh pelanggan di butik fisik untuk konsultasi perhiasan, pengukuran cincin, atau inspeksi berlian bersama spesialis.
- **_Dilarang digunakan (Do not use):_** `Booking Toko`, `Antrean`, `Jadwal Ketemu`.

### 9. Inquiry
- **Definisi:** Permintaan informasi khusus, pertanyaan pengadaan batu khusus, atau permohonan konsultasi desain perhiasan kustom (*bespoke*) yang dikirimkan melalui formulir digital.
- **_Dilarang digunakan (Do not use):_** `Pesan Masuk`, `Chat`, `Kontak Kami`.

### 10. Cart
- **Definisi:** Wadah penampung sementara karya perhiasan yang dipilih oleh pelanggan sebelum melanjutkan ke tahap pembayaran. Mendukung pengunjung tamu (*guest*) dan pengguna terdaftar.
- **_Dilarang digunakan (Do not use):_** `Troli`, `Keranjang Belanja Biasa`.

### 11. Order
- **Definisi:** Kontrak transaksi pemesanan resmi yang tercatat dalam sistem setelah pelanggan menyelesaikan formulir checkout, berisi data pemesan, alamat, total harga, dan referensi pembayaran.
- **_Dilarang digunakan (Do not use):_** `Purchase`, `Nota`, `Faktur Sementara`.

### 12. Order Line
- **Definisi:** Baris rincian individual di dalam sebuah Order yang mencatat Piece, varian ukuran, kuantitas, serta harga satuan yang dikunci (*price snapshot*) pada saat transaksi.
- **_Dilarang digunakan (Do not use):_** `Sub-order`, `Item Pesanan`.

### 13. Wishlist
- **Definisi:** Daftar kurasi perhiasan favorit yang disimpan oleh pengguna terdaftar untuk ditinjau kembali di kemudian hari.
- **_Dilarang digunakan (Do not use):_** `Favorit`, `Bookmark`.

---

## 2. Definisi Status Sistem (Status Definitions)

Semua status di bawah ini adalah nilai enum resmi yang digunakan di tingkat database PostgreSQL, skema Prisma, kontrak API, dan logika antarmuka.

### A. Status Pesanan (Order Status)

```
pending_payment ──► paid ──► processing ──► shipped ──► completed
       │              │
       ▼              ▼
   cancelled       refunded
```

| Nilai Enum | Kategori | Arti & Kondisi Pemicu |
|---|---|---|
| `pending_payment` | Initial | Pesanan telah dibuat di checkout, menunggu penyelesaian pembayaran oleh pelanggan. |
| `paid` | Transisi | Pembayaran terverifikasi lunas (baik otomatis via Midtrans maupun diverifikasi manual oleh admin). |
| `processing` | Operasional | Tim atelier sedang menyiapkan perhiasan, melakukan inspeksi kualitas, atau proses *made-to-order*. |
| `shipped` | Logistik | Paket perhiasan telah diserahkan ke jasa kurir dengan asuransi pengiriman penuh; resi terbit. |
| `completed` | Terminal | Paket telah diterima oleh pelanggan dengan konfirmasi pengiriman berhasil. |
| `cancelled` | Terminal | Pesanan dibatalkan (karena kadaluarsa pembayaran, pembatalan sistem, atau permintaan pelanggan). |
| `refunded` | Terminal | Dana pesanan telah dikembalikan penuh atau sebagian kepada pelanggan. |

### B. Status Pembayaran (Payment Status) & Metode

**Metode Pembayaran (`PaymentMethod`):**
- `midtrans_snap`: Pembayaran digital terintegrasi (Virtual Account, QRIS, GoPay, Kartu Kredit).
- `manual_transfer`: Pembayaran via transfer rekening bank resmi butik dengan verifikasi manual admin.

**Status Pembayaran (`PaymentStatus`):**
| Nilai Enum | Kategori | Arti & Kondisi Pemicu |
|---|---|---|
| `pending` | Initial | Transaksi dibuat di payment gateway atau menunggu transfer oleh pembeli. |
| `awaiting_verification` | Khusus Manual | Bukti transfer telah diunggah oleh pembeli, menunggu konfirmasi tim finance/admin di panel admin. |
| `paid` | Sukses | Pembayaran berhasil diverifikasi secara penuh. |
| `failed` | Gagal | Transaksi ditolak oleh bank atau kartu kredit gagal diproses. |
| `expired` | Terminal Gagal | Batas waktu pembayaran (time-to-live) telah terlewati tanpa pelunasan. |

### C. Status Janji Temu (Appointment Status)

| Nilai Enum | Kategori | Arti & Kondisi Pemicu |
|---|---|---|
| `requested` | Initial | Tamu telah memilih slot waktu melalui website, menunggu konfirmasi pihak concierge. |
| `confirmed` | Aktif | Tim butik telah mengonfirmasi ketersediaan jadwal; email undangan terkirim ke tamu. |
| `completed` | Sukses | Tamu telah hadir di butik dan sesi konsultasi telah selesai dilaksanakan. |
| `cancelled` | Batal | Janji temu dibatalkan oleh tamu atau pihak butik sebelum waktu yang dijadwalkan. |
| `no_show` | Tidak Hadir | Tamu tidak hadir pada waktu yang telah disepakati tanpa pemberitahuan sebelumnya. |

### D. Ketersediaan Perhiasan (Piece Availability)

| Nilai Enum | Arti & Perilaku di Antarmuka |
|---|---|
| `in_stock` | Perhiasan siap kirim segera dari brankas butik. Tombol "Add to Bag" aktif. |
| `made_to_order` | Perhiasan akan ditempa khusus oleh pengrajin (waktu pembuatan 2–4 minggu). Tombol "Order to Make" aktif. |
| `reserved` | Perhiasan sedang direservasi khusus untuk sesi janji temu atau pelanggan tertentu. |
| `sold_out` | Perhiasan telah terjual habis dan saat ini tidak dapat dipesan online. |

### E. Asal Berlian (Diamond Origin)

| Nilai Enum | Arti & Filosofi |
|---|---|
| `natural` | Berlian alami hasil pembentukan bumi miliaran tahun lalu, diekstraksi secara etis, bersertifikat GIA/IGI. |
| `lab_grown` | Berlian hasil kreasi laboratorium berteknologi HPHT/CVD dengan struktur atom identik 100%, bersertifikat IGI/GIA. |

---

## 3. Peran Pengguna & Otorisasi (Roles)

| Peran (Role) | Hak Akses & Kewenangan dalam Sistem |
|---|---|
| `super_admin` | Akses penuh tanpa batas ke seluruh modul admin, manajemen staf admin, konfigurasi rahasia sistem, pengaturan payment gateway, dan audit log. |
| `admin` | Akses operasional panel admin: mengelola katalog produk, data berlian, memverifikasi transfer bank manual, memperbarui status pesanan, dan mengelola kalender janji temu butik. |
| `customer` | Pengguna terdaftar yang memiliki akun login: dapat melihat riwayat pesanan pribadi, mengelola buku alamat pengiriman, menyimpan wishlist, dan memperbarui profil. |
| `guest` | Pengunjung tanpa akun yang dapat menjelajahi seluruh katalog publik, melakukan booking janji temu, dan menyelesaikan checkout pesanan tanpa harus mendaftar akun. |

---

## 4. Istilah Bisnis Perhiasan (Business Terms)

- **One-of-a-Kind:** Karya perhiasan eksklusif yang hanya diciptakan satu buah di dunia, sering kali berpusat pada berlian alami berkarakteristik unik.
- **Made-to-Order:** Proses pembuatan perhiasan yang baru dimulai setelah pesanan diterima dan dikonfirmasi, disesuaikan dengan ukuran jari atau preferensi logam pelanggan.
- **Provenance:** Catatan silsilah, transparansi asal-usul, dan ketelusuran etis batu permata dari sumber tambang atau laboratorium hingga menjadi perhiasan jadi.
- **Consignment:** Skema penitipan perhiasan koleksi istimewa di bawah kurasi dan standar mutu Bifrost Jewellery.
- **Ring Size:** Standar pengukuran keliling jari pemakai, menggunakan skala metrik standar Hong Kong / US yang disediakan di panduan ukuran (*Ring Size Guide*).
- **Complimentary Resizing:** Layanan penyesuaian ukuran cincin gratis satu kali bagi pelanggan dalam kurun waktu 30 hari pasca-pembelian.
- **Insurance-Grade Appraisal:** Dokumen penaksiran nilai resmi yang diterbitkan untuk keperluan asuransi perhiasan bernilai tinggi.
- **Certificate Match:** Proses verifikasi kecocokan nomor inskripsi laser mikro pada sabuk berlian (*girdle*) dengan nomor fisik laporan grading GIA atau IGI.

---

## 5. Singkatan Teknis (Technical Abbreviations)

| Singkatan | Kepanjangan | Konteks Penggunaan |
|---|---|---|
| **GIA** | Gemological Institute of America | Lembaga otoritas grading berlian paling prestisius di dunia. |
| **IGI** | International Gemological Institute | Lembaga grading internasional terkemuka (standar utama untuk lab-grown diamonds). |
| **4Cs** | Carat, Cut, Color, Clarity | Empat pilar penilaian mutu berlian internasional. |
| **TBD** | To Be Determined (`<TBD: ...>`) | Penanda standar untuk data bisnis yang belum ditentukan dan wajib dilengkapi. |
| **SKU** | Stock Keeping Unit | Pengidentifikasi unik alfanumerik untuk setiap varian fisik perhiasan. |
| **RBAC** | Role-Based Access Control | Mekanisme pembatasan hak akses sistem berdasarkan peran pengguna. |
| **CTA** | Call to Action | Elemen tombol atau aksi utama pada antarmuka pengguna. |
| **ISR** | Incremental Static Regeneration | Strategi rendering Next.js untuk memperbarui halaman statis secara berkala di latar belakang. |
| **SSG** | Static Site Generation | Pembuatan halaman HTML statis pada saat proses build aplikasi. |
| **JSON-LD** | JavaScript Object Notation for Linked Data | Format data terstruktur untuk mesin pencari Google (schema.org). |
| **VAT / PPN** | Value Added Tax / Pajak Pertambahan Nilai | Komponen pajak resmi yang berlaku pada transaksi ritel di Indonesia. |
| **IDR** | Indonesian Rupiah | Mata uang resmi Republik Indonesia (mata uang transaksi utama). |
| **USD** | United States Dollar | Mata uang Dolar Amerika Serikat (tampilan perkiraan kurs untuk audiens global). |

---

## 6. Dokumen Terkait

- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) — Identitas dan lingkup bisnis proyek
- [DATABASE.md](./DATABASE.md) — Definisi skema tabel dan implementasi enum Prisma
- [API_CONTRACT.md](./API_CONTRACT.md) — Kontrak endpoint dan validasi status
- [UI_UX.md](./UI_UX.md) — Panduan tone of voice dan aturan copywriting
