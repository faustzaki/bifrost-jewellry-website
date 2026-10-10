# BUGS.md — Log Pelaporan & Pelacakan Bug Sistem

> **Tujuan:** Dokumen ini digunakan untuk mencatat, melacak, dan mengelola perbaikan kesalahan (*bugs*) atau anomali teknis yang ditemukan pada sistem Bifrost Jewellery. Setiap bug wajib dilaporkan menggunakan template terstandarisasi dengan tingkat keparahan (*severity*) yang jelas.

---

## 1. Skala Tingkat Keparahan Bug (Severity Levels)

| Tingkat Keparahan | Klasifikasi | Kriteria & Dampak Sistem | Target SLA Perbaikan |
|---|---|---|---|
| **P0 — Blocker** | Kritis / Darurat | Sistem lumpuh total, kegagalan transaksi pembayaran, kalkulasi harga salah, data korup, atau celah keamanan finansial. | Segera diperbaiki (< 4 jam) |
| **P1 — Critical** | Sangat Tinggi | Fitur utama terputus (misalnya: gagal memuat katalog, tombol checkout macet, notifikasi webhook gagal diproses), tidak ada workaround. | < 24 jam |
| **P2 — Major** | Sedang | Fitur sekunder mengalami anomali (misalnya: filter 4Cs tidak akurat untuk bentuk tertentu, layout rusak pada mobile), ada workaround sementara. | < 3 hari kerja |
| **P3 — Minor** | Rendah | Anomali visual kosmetik, kesalahan tipografi kecil pada copy, ketidaksesuaian margin/padding ringan. | Rilis terjadwal berikutnya |

---

## 2. Template Pelaporan Bug Standar

Gunakan format di bawah ini saat menambahkan bug baru:

```markdown
### [BUG-NNN] Judul Bug yang Ringkas & Jelas

- **Severity:** [P0 / P1 / P2 / P3]
- **Komponen / Halaman Terkait:** [contoh: `/checkout`, `/pieces/[slug]`, `src/lib/money.ts`]
- **Lingkungan Ditemukan:** [Local / Preview / Production]
- **Tanggal Ditemukan:** YYYY-MM-DD
- **Pelapor:** [Nama / AI Agent]

#### Langkah-Langkah Reproduksi (Steps to Reproduce):
1. Buka halaman ...
2. Masukkan input ...
3. Klik tombol ...

#### Perilaku Aktual (Actual Behavior):
Deskripsikan apa yang terjadi saat bug muncul (beserta pesan error atau screenshot jika ada).

#### Perilaku yang Diharapkan (Expected Behavior):
Deskripsikan apa yang seharusnya terjadi sesuai spesifikasi dokumen `.AGENTS/`.

#### Analisis Akar Masalah (Root Cause Analysis - RCA):
Penjelasan teknis penyebab terjadinya kesalahan (diisi oleh pengembang saat investigasi).

#### Rencana & Status Perbaikan (Fix Plan & Status):
- [ ] Analisis & reproduksi lokal
- [ ] Penulisan pengujian yang gagal (*failing test*)
- [ ] Implementasi perbaikan kode
- [ ] Verifikasi kelulusan test & PR merge
```

---

## 3. Daftar Bug Aktif (Active Bugs)

*(Saat ini tidak ada bug aktif. Proyek masih berada dalam Fase 1: Perencanaan & Dokumentasi Fondasi.)*

---

## 4. Log Bug Terselesaikan (Resolved Bugs)

| ID Bug | Judul Singkat | Severity | Tanggal Selesai | Commit / PR |
|---|---|---|---|---|
| *Belum ada* | — | — | — | — |

---

## 5. Dokumen Terkait

- [CODING_RULES.md](../CODING_RULES.md) — Standar penulisan kode untuk mencegah regresi
- [TASKS/TODO.md](./TODO.md) — Daftar tugas operasional
- [CURRENT_STATE.md](../CURRENT_STATE.md) — Status memori terkini proyek
