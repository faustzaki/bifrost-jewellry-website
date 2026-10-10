# 🎨 PATTERN — Pemisahan Komponen RSC & Client Island Quiet Luxury (COMPONENT_PATTERN)

- **Applies To:** Frontend / UI / React 19 / Next.js 15
- **Framework:** Next.js 15 (App Router), Tailwind CSS v4, Framer Motion, Phosphor Icons
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Seluruh perancangan antarmuka visual pengguna di direktori `src/components/` dan `src/app/**`.
- Memisahkan komponen pembungkus server (*React Server Component / RSC*) yang bertugas mengambil data langsung dari basis data tanpa JavaScript bundle klien, dengan pulau interaktif (*Client Island*) yang menangani event peramban, state interaksi, animasi, atau akses API Web.
- Membangun antarmuka bertema *quiet luxury*: tata letak lapang, kontras warna AA/AAA, transisi mikro halus, serta aksesibilitas keyboard menyeluruh.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Komponen konten presentasi murni yang tidak memiliki interaksi pengguna (misal: kartu statis, footer, daftar klausul teks) — pertahankan komponen tersebut sebagai **Server Component murni** tanpa menambahkan direktif `'use client'`.
- Menggunakan pustaka state global yang berat (seperti Redux/Zustand) untuk hal-hal yang dapat dikelola dengan native URL Search Params atau local React state.

---

## 3. Struktur Blueprint (Structure)

```
src/
└── components/
    └── pieces/
        ├── product-card.tsx           # Server Component (Pengambilan data & markup dasar)
        ├── product-gallery.tsx        # Client Island ('use client', lightbox, zoom, drag)
        ├── ring-size-selector.tsx     # Client Island ('use client', pemilihan ukuran, modal guide)
        └── add-to-bag-button.tsx      # Client Island ('use client', Server Action trigger, loading state)
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah contoh nyata implementasi kartu perhiasan (*Product Card*) dan pulau interaktif tombol aksi hibrida (*Dual Action Client Island*), menggunakan token semantik Tailwind v4 `--ql-*` dan kepatuhan aksesibilitas WCAG AA:

### 1. Server Component Induk (`src/components/pieces/product-card.tsx`)

```tsx
import Image from 'next/image';
import Link from 'next/link';
import { formatMoney } from '@/lib/money';
import { QuickActionIsland } from './quick-action-island';

export interface ProductCardProps {
  piece: {
    id: string;
    slug: string;
    sku: string;
    name: string;
    metal: string;
    priceIdrMinor: bigint | number;
    priceUsdMinor: number;
    availability: 'in_stock' | 'made_to_order' | 'reserved' | 'sold_out';
    primaryImage: string;
    secondaryImage?: string | null;
    diamondCarat?: number | null;
    diamondShape?: string | null;
  };
  preferredCurrency?: 'IDR' | 'USD';
}

/**
 * Server Component: Merender kartu perhiasan tanpa bundle JS interaktif berat.
 * Mengonsumsi token desain Tailwind v4 dan Next.js Image teroptimasi.
 */
export function ProductCard({ piece, preferredCurrency = 'IDR' }: ProductCardProps) {
  const formattedPrice = formatMoney(
    preferredCurrency === 'IDR' ? piece.priceIdrMinor : piece.priceUsdMinor,
    preferredCurrency
  );

  return (
    <article className="group relative flex flex-col bg-[var(--ql-surface)] border border-[var(--ql-line)] transition-colors duration-300 hover:border-[var(--ql-ink-muted)]">
      {/* Container Gambar Rasio 4:5 Vertikal Mewah */}
      <Link
        href={`/pieces/${piece.slug}`}
        className="relative block aspect-[4/5] w-full overflow-hidden bg-[var(--ql-canvas)]"
        aria-label={`Lihat detail ${piece.name}`}
      >
        <Image
          src={piece.primaryImage}
          alt={piece.name}
          fill
          sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
          className="object-cover object-center transition-transform duration-700 ease-out group-hover:scale-105"
          priority={false}
        />

        {/* Gambar Sekunder Halus saat Hover (Jika Tersedia) */}
        {piece.secondaryImage && (
          <Image
            src={piece.secondaryImage}
            alt={`${piece.name} sudut detail`}
            fill
            sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
            className="object-cover object-center opacity-0 transition-opacity duration-500 ease-out group-hover:opacity-100"
          />
        )}

        {/* Lencana Ketersediaan Elegan */}
        {piece.availability !== 'in_stock' && (
          <span className="absolute top-3 left-3 bg-[var(--ql-canvas)]/90 backdrop-blur-xs px-2.5 py-1 text-xs uppercase tracking-widest text-[var(--ql-ink-muted)] border border-[var(--ql-line)]">
            {piece.availability === 'made_to_order' ? 'Made to Order' : piece.availability}
          </span>
        )}
      </Link>

      {/* Rincian Deskripsi Editorial */}
      <div className="flex flex-1 flex-col p-5">
        <div className="mb-1 flex items-baseline justify-between text-xs tracking-wider text-[var(--ql-ink-muted)] uppercase">
          <span>{piece.metal.replace('_', ' ')}</span>
          {piece.diamondCarat && (
            <span className="font-mono text-[var(--ql-ink)]">
              {piece.diamondCarat} ct {piece.diamondShape}
            </span>
          )}
        </div>

        <h3 className="font-serif text-lg font-normal text-[var(--ql-ink)] transition-colors group-hover:text-[var(--ql-gold-deep)]">
          <Link href={`/pieces/${piece.slug}`} className="focus:outline-hidden focus-visible:underline">
            {piece.name}
          </Link>
        </h3>

        <p className="mt-2 text-sm font-medium text-[var(--ql-ink)]">
          {formattedPrice}
        </p>

        {/* Client Island: Interaksi Cepat Pembelian / Reservasi */}
        <div className="mt-4 pt-3 border-t border-[var(--ql-line)]">
          <QuickActionIsland pieceId={piece.id} pieceSlug={piece.slug} availability={piece.availability} />
        </div>
      </div>
    </article>
  );
}
```

### 2. Client Island Interaktif (`src/components/pieces/quick-action-island.tsx`)

```tsx
'use client';

import { useState, useTransition } from 'react';
import { useRouter } from 'next/navigation';
import { ArrowUpRight, Sparkle } from '@phosphor-icons/react';

interface QuickActionIslandProps {
  pieceId: string;
  pieceSlug: string;
  availability: 'in_stock' | 'made_to_order' | 'reserved' | 'sold_out';
}

/**
 * Client Island: Menangani state interaksi pengguna, event tombol, dan navigasi.
 * Dilengkapi animasi micro-interaction dan feedback visual.
 */
export function QuickActionIsland({ pieceId, pieceSlug, availability }: QuickActionIslandProps) {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();
  const [feedback, setFeedback] = useState<string | null>(null);

  const handleInquire = () => {
    startTransition(() => {
      router.push(`/consultation?pieceId=${encodeURIComponent(pieceId)}`);
    });
  };

  const handleBookViewing = () => {
    startTransition(() => {
      router.push(`/appointments?pieceId=${encodeURIComponent(pieceId)}`);
    });
  };

  if (availability === 'sold_out' || availability === 'reserved') {
    return (
      <button
        type="button"
        onClick={handleInquire}
        disabled={isPending}
        className="w-full flex items-center justify-center gap-1.5 py-2 text-xs tracking-wider uppercase text-[var(--ql-ink-muted)] hover:text-[var(--ql-ink)] transition-colors focus-visible:ring-1 focus-visible:ring-[var(--ql-ink)] outline-hidden cursor-pointer"
      >
        <span>Request Similar Piece</span>
        <ArrowUpRight size={14} weight="regular" />
      </button>
    );
  }

  return (
    <div className="flex items-center gap-2">
      <button
        type="button"
        onClick={() => router.push(`/pieces/${pieceSlug}`)}
        className="flex-1 py-2 text-xs tracking-wider uppercase text-[var(--ql-ink)] bg-transparent border border-[var(--ql-line)] hover:border-[var(--ql-ink)] transition-colors focus-visible:ring-1 focus-visible:ring-[var(--ql-ink)] outline-hidden cursor-pointer text-center"
      >
        View Piece
      </button>

      <button
        type="button"
        onClick={handleBookViewing}
        disabled={isPending}
        aria-label="Jadwalkan kunjungan butik privat untuk karya ini"
        className="px-3 py-2 text-[var(--ql-ink-muted)] hover:text-[var(--ql-gold-deep)] border border-[var(--ql-line)] hover:border-[var(--ql-gold-deep)] transition-colors focus-visible:ring-1 focus-visible:ring-[var(--ql-gold-deep)] outline-hidden cursor-pointer"
        title="Book Boutique Viewing"
      >
        <Sparkle size={14} weight="light" />
      </button>
    </div>
  );
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

```tsx
// ❌ ANTI-PATTERN 1: Menjadikan seluruh halaman atau komponen induk sebagai 'use client'
'use client'; // DILARANG jika halaman hanya memuat daftar produk dari basis data!
export default async function CatalogPage() {
  // Mengakibatkan seluruh bundle database client terkirim ke peramban dan merusak SSR/SEO
}

// ❌ ANTI-PATTERN 2: Menggunakan warna heksadesimal arbitrary langsung di komponen
<div className="bg-[#FAF8F5] text-[#1A1A1A] border-[#E7E2DA]">
  {/* DILARANG: Melanggar aturan token semantik --ql-* dan menyulitkan pemeliharaan tema */}
</div>

// ❌ ANTI-PATTERN 3: Menghilangkan outline fokus aksesibilitas tanpa pengganti
<button className="outline-none focus:outline-none">
  {/* DILARANG: Merusak navigasi keyboard pengguna tunanetra / disabilitas motorik */}
</button>
```

---

## 6. Aturan & Invariants (Rules)

1. **Server Component Default:** Seluruh komponen secara default adalah React Server Components. Tambahkan direktif `'use client'` hanya pada berkas daun (*leaf nodes*) yang membutuhkan event handler atau hooks.
2. **Konsumsi Token Semantik Murni:** Gunakan token warna semantik yang didefinisikan di `UI_UX.md`:
   - Latar: `bg-[var(--ql-canvas)]` atau `bg-[var(--ql-surface)]`
   - Teks Utama: `text-[var(--ql-ink)]` (kontras 16.42:1 ✅)
   - Teks Sekunder: `text-[var(--ql-ink-muted)]` (kontras 5.63:1 ✅)
   - Garis Batas: `border-[var(--ql-line)]`
   - Aksen Emas Dalam: `text-[var(--ql-gold-deep)]` (kontras 5.42:1 ✅)
3. **Aksesibilitas Keyboard Penuh:** Setiap elemen interaktif (`button`, `a`, `input`) wajib memiliki visual focus ring yang jelas (`focus-visible:ring-1` atau `focus-visible:underline`).
4. **Optimasi Gambar Wajib:** Gunakan `next/image` dengan atribut `sizes`, `alt` deskriptif bahasa manusia, dan larangan memuat gambar tanpa batas dimensi.
5. **Quiet Luxury Restraint:** Hindari elemen animasi berlebihan, tombol berkedip, teks berteriak, atau pop-up agresif.

---

## 7. Pola & Dokumen Terkait

- [UI_UX.md](../UI_UX.md) — Panduan token warna, tipografi, dan checklist aksesibilitas
- [CODING_RULES.md](../CODING_RULES.md) — Standar kode antarmuka Next.js
