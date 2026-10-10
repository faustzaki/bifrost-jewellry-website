# MANUAL.md — Panduan Tata Kelola & Pemeliharaan Dokumentasi `.AGENTS/`

> **Tujuan:** Dokumen ini memberikan panduan operasional bagi tim pengembang dan AI agent dalam menggunakan, memelihara, dan mengembangkan struktur dokumentasi di dalam folder `.AGENTS/`. Folder ini berfungsi sebagai memori persisten dan pusat pengetahuan arsitektural proyek Bifrost Jewellery.

---

## 1. Mengapa Folder `.AGENTS/` Dibutuhkan?

Dalam pengembangan perangkat lunak modern yang didukung AI agent:
1. **Zero Context Loss:** AI agent sering kali mengalami kehilangan memori antar sesi. Folder `.AGENTS/` menyediakan konteks lengkap yang dapat dimuat kembali secara instan.
2. **Konsistensi Desain & Arsitektur:** Mencegah AI agent berhalusinasi, menciptakan pola kode yang saling bertentangan, atau menggunakan dependensi yang tidak disetujui.
3. **Single Source of Truth:** Seluruh keputusan arsitektur, skema basis data, token warna, dan istilah bisnis terdokumentasi rapi di satu tempat.

---

## 2. Struktur Hierarki Tier

Folder `.AGENTS/` dibagi menjadi 4 tingkatan hierarki (*Tier*):

```
.AGENTS/
├── Tier 0: CURRENT_STATE.md (Memori Sesi & Open Inputs)
├── Tier 1: AGENTS.md, PROJECT_CONTEXT.md, CODING_RULES.md, GLOSSARY.md
├── Tier 2: ARCHITECTURE.md, STACK.md, DATABASE.md, API_CONTRACT.md, UI_UX.md, MANUAL.md
└── Tier 3: DECISIONS/, TASKS/, PROMPTS/ (serta spesifikasi fitur pendukung)
```

- **Tier 0:** Berkas dinamis yang dibaca pertama kali pada awal sesi dan diperbarui setiap akhir sesi.
- **Tier 1:** Dokumen fondasi yang wajib dipahami sebelum menulis sebaris kode pun.
- **Tier 2:** Spesifikasi teknis mendalam yang dibuka sesuai domain tugas yang sedang dikerjakan.
- **Tier 3:** Berkas operasional, rekaman keputusan historis (ADR), daftar tugas harian, dan template prompt.

---

## 3. Aturan DO & DON'T (Tabel Praktik Terbaik)

| Praktik Wajib (DO) | Praktik Terlarang (DON'T) |
|---|---|
| **DO:** Selalu perbarui `CURRENT_STATE.md` di akhir setiap sesi kerja. | **DON'T:** Menutup sesi tanpa mencatat apa yang telah diselesaikan. |
| **DO:** Gunakan penanda `<TBD: ...>` jika fakta bisnis riil belum diketahui. | **DON'T:** Mengarang data palsu, sertifikat GIA fiktif, atau alamat dummy. |
| **DO:** Tulis seluruh dokumentasi internal dalam **Bahasa Indonesia**. | **DON'T:** Mencampuradukkan bahasa narasi dokumentasi secara acak. |
| **DO:** Konsultasikan `DECISIONS/` sebelum menyarankan perubahan arsitektur. | **DON'T:** Melanggar keputusan yang telah terkunci di ADR tanpa diskusi. |
| **DO:** Gunakan token warna semantik (`--ql-*`) dari `UI_UX.md`. | **DON'T:** Menuliskan nilai hex sembarangan (arbitrary values) di komponen. |
| **DO:** Buat ADR baru di `DECISIONS/` jika ada perubahan arsitektur besar. | **DON'T:** Mengubah arsitektur fundamental secara diam-diam. |

---

## 4. Tanya Jawab Umum (Frequently Asked Questions / FAQ)

### Q1: Mengapa dokumentasi AI ditulis dalam Bahasa Indonesia, padahal teks publik website berbahasa Inggris?
**Jawaban:** Berdasarkan keputusan arsitektur [ADR-008](./DECISIONS/008-english-only-content.md), teks publik storefront menggunakan Bahasa Inggris untuk membangun citra rumah perhiasan mewah internasional. Namun, dokumentasi sistem internal, aturan coding, dan instruksi AI agent ditulis dalam Bahasa Indonesia untuk memudahkan kolaborasi, kejelasan instruksi tim pengembang lokal, dan ketepatan evaluasi logika bisnis.

### Q2: Bagaimana jika saya ingin menambahkan pustaka npm baru?
**Jawaban:** Dilarang langsung menginstal paket baru. Periksa terlebih dahulu [STACK.md](./STACK.md). Jika pustaka yang ada (seperti Zod, Framer Motion, Lenis) belum mencakup kebutuhan, diskusikan terlebih dahulu dengan Lead Architect atau User. Jika disetujui, catat penambahan tersebut di `STACK.md` dan buat ADR jika dependensi tersebut bersifat fundamental.

### Q3: Bagaimana cara menangani nilai bisnis yang belum diketahui (seperti nomor telepon butik atau domain produksi)?
**Jawaban:** Jangan pernah mengarang data. Tuliskan penanda eksplisit `<TBD: ...>` (misalnya `<TBD: telepon butik>`) pada teks dokumen, lalu daftarkan penanda tersebut pada tabel `§Open Inputs` di [CURRENT_STATE.md](./CURRENT_STATE.md). Saat pemilik bisnis memberikan data riil, gantikan penanda tersebut di seluruh file dan hapus barisnya dari `CURRENT_STATE.md`.

### Q4: Apakah boleh menggunakan REST API untuk membuat produk baru di panel admin?
**Jawaban:** Tidak. Sesuai [ARCHITECTURE.md](./ARCHITECTURE.md) dan [CODING_RULES.md](./CODING_RULES.md), seluruh mutasi data di panel admin wajib menggunakan **Server Actions** Next.js yang divalidasi menggunakan skema Zod di sisi server. REST Route Handlers hanya diperuntukkan bagi endpoint publik baca (*read-only*), webhook Midtrans, dan API janji temu publik.

---

## 5. Dokumen Terkait

- [AGENTS.md](./AGENTS.md) — Router utama navigasi agent
- [CURRENT_STATE.md](./CURRENT_STATE.md) — Memori sesi dan pelacak TBD
- [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) — Identitas dan batasan proyek
- [DECISIONS/README.md](./DECISIONS/README.md) — Daftar rekaman keputusan arsitektur
