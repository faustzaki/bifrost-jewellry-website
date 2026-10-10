# 🏛️ FEATURE — Lokasi Butik Fisik (BOUTIQUE)

> **Status:** Planned  
> **Scope:** Public Boutique Showcase & Store Locator  
> **Primary Goal:** Menampilkan profil, alamat, peta interaktif, jam operasional, dan kontak resmi butik fisik Bifrost Jewellery guna memperkuat kredibilitas butik serta memudahkan kunjungan tatap muka para klien.

---

## 1. Ringkasan (Overview)

Bifrost Jewellery menghadirkan ruang butik fisik bernuansa tenang dan intim (*private salon sanctuary*) yang dirancang khusus untuk memamerkan koleksi *high jewellery* dan berlian pilihan. Halaman Butik (`/boutiques`) menyajikan informasi komprehensif mengenai galeri fisik butik, lokasi tepat di pusat perbelanjaan atau distrik prestisius, jam buka layanan, fasilitas valet, serta tombol aksi langsung untuk memesan janji temu pribadi di butik tersebut.

Tampilan dirancang dengan estetika arsitektural mewah: fotografi ruang butik beresolusi tinggi, peta interaktif terintegrasi yang responsif, dan petunjuk arah navigasi Google Maps.

---

## 2. Kebutuhan Pengguna (User Stories)

- **Sebagai Klien yang Ingin Berkunjung:** Saya ingin mengetahui alamat lengkap, lantai butik, dan jam operasional butik fisik Bifrost Jewellery agar saya dapat merencanakan kunjungan dengan tepat.
- **Sebagai Pengemudi Kendaraan:** Saya ingin melihat petunjuk akses parkir atau layanan valet butik serta membuka peta navigasi Google Maps dengan satu klik dari ponsel pintar saya.
- **Sebagai Pelanggan Butik:** Saya ingin menghubungi nomor telepon resmi atau concierge WhatsApp butik untuk menanyakan ketersediaan staf sebelum saya melangkah menuju lokasi.
- **Sebagai Calon Pengantin:** Saya ingin mengklik tombol "Book an Appointment at this Boutique" langsung dari profil butik yang saya tuju.

---

## 3. Aturan Bisnis (Business Rules)

1. **Status Butik Aktif (`is_active`):**
   - Hanya butik dengan status `is_active = true` yang ditampilkan pada halaman direktori publik dan dapat dipilih pada alur pemesanan janji temu.
2. **Keterbukaan Informasi Kontak:**
   - Setiap profil butik wajib menyertakan jam buka operasional hari kerja dan akhir pekan, nomor telepon resmi butik, dan tautan peta navigasi (*direct map link*).
   - Nilai riil yang belum difinalisasi oleh pihak manajemen wajib ditandai menggunakan marker eksplisit:
     - Alamat Fisik Butik: `<TBD: alamat fisik butik Bifrost Jewellery>`
     - Jam Operasional: `<TBD: jam buka operasional butik>`
     - Nomor Telepon Kontak: `<TBD: nomor telepon resmi butik>`
     - Tautan Peta Google Maps: `<TBD: link Google Maps butik>`
3. **Integrasi Skema SEO Terstruktur (LocalBusiness Schema):**
   - Setiap entitas butik wajib merender script JSON-LD berstandar schema.org `JewelryStore` / `LocalBusiness` agar terindeks sempurna di mesin pencari Google Maps dan SERP lokal.

---

## 4. Integrasi Data (DATABASE.md)

Modul ini membaca data dari tabel:

- `boutiques`: Entitas data butik (`id`, `name`, `slug`, `address_line`, `city`, `province_or_state`, `postal_code`, `map_url`, `phone`, `email`, `opening_hours`, `image_url`, `is_active`, `display_order`).
- `appointments`: Relasi pemesanan janji temu yang terikat pada butik ini.

---

## 5. Endpoints & Server Actions (API_CONTRACT.md)

### Read Operations (Server Component)

- Halaman `/boutiques` di-render sebagai React Server Component yang mengambil data langsung via Prisma ORM:
  ```typescript
  const boutiques = await prisma.boutique.findMany({
    where: { isActive: true },
    orderBy: { displayOrder: 'asc' }
  });
  ```
- **`GET /api/boutiques` (Public REST Endpoint):**
  - **Respon 200 OK:**
    ```json
    {
      "success": true,
      "data": [
        {
          "id": "b1a2b3c4-0000-0000-0000-111122223333",
          "name": "Bifrost Flagship Boutique",
          "slug": "flagship-boutique-jakarta",
          "addressLine": "<TBD: alamat fisik butik Bifrost Jewellery>",
          "city": "Jakarta Selatan",
          "phone": "<TBD: nomor telepon resmi butik>",
          "openingHours": "<TBD: jam buka operasional butik>",
          "mapUrl": "<TBD: link Google Maps butik>",
          "imageUrl": "https://blob.bifrostjewellery.com/boutiques/flagship-interior.webp"
        }
      ]
    }
    ```

---

## 6. Komponen Antarmuka Pengguna (UI_UX.md)

- `BoutiqueCard`: Kartu horizontal atau vertikal elegan yang memadukan foto interior butik (rasio 16:9 atau 4:5), alamat lengkap, jam operasional, dan dua tombol aksi: "View on Map ↗" dan "Book Appointment".
- `InteractiveMapEmbed`: Peta interaktif yang dimuat secara asinkron (*lazy-loaded iframe*) dengan styling monokrom tenang yang selaras dengan tema quiet luxury (tanpa warna neon cerah).
- `ConciergeContactBlock`: Baris informasi kontak dengan ikon Phosphor halus untuk panggilan telepon, chat WhatsApp concierge, dan email resmi.
- `LocalBusinessJsonLd`: Komponen skrip semantik JSON-LD untuk optimasi SEO pencarian lokal Google.

---

## 7. Kriteria Penerimaan (Acceptance Criteria)

- [ ] Halaman `/boutiques` merender seluruh butik aktif dengan informasi alamat, jam operasional, dan nomor kontak secara lengkap.
- [ ] Mengklik tautan "View on Map" membuka Google Maps di tab baru dengan koordinat lokasi yang tepat.
- [ ] Mengklik "Book Appointment" pada kartu butik tertentu mengarahkan pengguna ke alur `/appointments` dengan parameter butik tersebut langsung terpilih.
- [ ] Skrip JSON-LD `JewelryStore` tervalidasi 100% valid tanpa peringatan pada Google Rich Results Test Tool.
- [ ] Tampilan peta interaktif menerapkan *lazy loading* sehingga tidak menurunkan skor Lighthouse Performance halaman.

---

## 8. Dokumen Terkait

- [APPOINTMENT.md](./APPOINTMENT.md) — Alur pemesanan jadwal kunjungan ke butik
- [LEGAL_AND_FAQ.md](./LEGAL_AND_FAQ.md) — FAQ seputar kunjungan dan protokol butik
- [UI_UX.md](../UI_UX.md) — Pedoman visual dan tipografi kartu butik
