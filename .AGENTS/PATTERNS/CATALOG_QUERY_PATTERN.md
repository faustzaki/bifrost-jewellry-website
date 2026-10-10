# 💎 PATTERN — Query Builder Dinamis Katalog & Filter Berlian (CATALOG_QUERY_PATTERN)

- **Applies To:** Backend / Data Layer / Database Queries
- **Framework:** Prisma ORM, PostgreSQL 16, TypeScript (Strict)
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Membangun kueri pencarian dan penyaringan data dinamis di halaman katalog koleksi perhiasan (`/collections`) dan mesin filter berlian 4Cs (`/diamonds`).
- Menangani kombinasi multi-filter yang kompleks (asal batu berlian: Natural vs Lab-Grown, multi-bentuk: Round/Oval/Emerald, rentang karat, rentang harga, material logam, dan sorting) secara aman tanpa risiko SQL Injection.
- Mengimplementasikan penomoran halaman (*pagination*) yang efisien menggunakan offset terukur atau cursor pagination pada set data inventaris besar, selaras dengan indeks komposit basis data.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Kueri data tunggal berdasarkan primary key atau unik slug — gunakan langsung `prisma.piece.findUnique()`.
- Menggunakan concatenation string SQL mentah (`prisma.$queryRawUnsafe`) — dilarang keras karena membuka celah keamanan injeksi SQL.

---

## 3. Struktur Blueprint (Structure)

```
src/
└── lib/
    └── db/
        ├── queries/
        │   ├── catalog-query.ts       # Query builder katalog & berlian
        │   └── types.ts               # Tipe input filter dan hasil
        └── prisma.ts                  # Client singleton Prisma
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah implementasi fungsi `queryCatalogPieces` lengkap dan teroptimasi menggunakan Prisma ORM bertipe ketat:

```typescript
// src/lib/db/queries/catalog-query.ts
import { Prisma, DiamondOrigin, DiamondShape, DiamondCut, MetalType, PieceAvailability } from '@prisma/client';
import { prisma } from '@/lib/db';

export interface CatalogFilterParams {
  categorySlug?: string;
  collectionSlug?: string;
  metal?: MetalType;
  origin?: DiamondOrigin;
  shapes?: DiamondShape[];
  cuts?: DiamondCut[];
  caratMin?: number;
  caratMax?: number;
  priceMinMinor?: number | bigint;
  priceMaxMinor?: number | bigint;
  currency?: 'IDR' | 'USD';
  availability?: PieceAvailability;
  searchQuery?: string;
  sortBy?: 'newest' | 'price_asc' | 'price_desc' | 'carat_desc';
  page?: number;
  limit?: number;
}

export interface CatalogQueryResult {
  pieces: Array<{
    id: string;
    slug: string;
    sku: string;
    name: string;
    metal: MetalType;
    priceIdrMinor: bigint;
    priceUsdMinor: number;
    availability: PieceAvailability;
    primaryImage: string | null;
    secondaryImage: string | null;
    diamond?: {
      origin: DiamondOrigin;
      carat: number;
      shape: DiamondShape;
      cut: DiamondCut;
      colorGrade: string;
      clarityGrade: string;
    } | null;
  }>;
  pagination: {
    page: number;
    limit: number;
    totalItems: number;
    totalPages: number;
    hasNextPage: boolean;
  };
}

/**
 * Membangun dan mengeksekusi kueri katalog perhiasan dan filter berlian 4Cs secara aman.
 */
export async function queryCatalogPieces(params: CatalogFilterParams): Promise<CatalogQueryResult> {
  const page = Math.max(1, params.page ?? 1);
  const limit = Math.min(48, Math.max(1, params.limit ?? 12));
  const skip = (page - 1) * limit;

  // 1. Bangun klausa WHERE secara dinamis dan tipe aman
  const where: Prisma.PieceWhereInput = {
    isPublished: true, // Hanya tampilkan karya yang diterbitkan
  };

  if (params.categorySlug) {
    where.category = { slug: params.categorySlug };
  }

  if (params.collectionSlug) {
    where.collection = { slug: params.collectionSlug };
  }

  if (params.metal) {
    where.metal = params.metal;
  }

  if (params.availability) {
    where.availability = params.availability;
  }

  // Filter Harga berdasarkan mata uang aktif
  if (params.priceMinMinor !== undefined || params.priceMaxMinor !== undefined) {
    if (params.currency === 'USD') {
      where.priceUsdMinor = {};
      if (params.priceMinMinor !== undefined) where.priceUsdMinor.gte = Number(params.priceMinMinor);
      if (params.priceMaxMinor !== undefined) where.priceUsdMinor.lte = Number(params.priceMaxMinor);
    } else {
      where.priceIdrMinor = {};
      if (params.priceMinMinor !== undefined) where.priceIdrMinor.gte = BigInt(params.priceMinMinor);
      if (params.priceMaxMinor !== undefined) where.priceIdrMinor.lte = BigInt(params.priceMaxMinor);
    }
  }

  // Filter Pencarian Teks Bebas (Nama / SKU)
  if (params.searchQuery && params.searchQuery.trim().length > 0) {
    const term = params.searchQuery.trim();
    where.OR = [
      { name: { contains: term, mode: 'insensitive' } },
      { sku: { contains: term, mode: 'insensitive' } },
    ];
  }

  // 2. Filter Khusus Batu Berlian (Relasi Diamonds)
  const diamondConditions: Prisma.DiamondWhereInput = {};

  if (params.origin) {
    diamondConditions.origin = params.origin;
  }

  if (params.shapes && params.shapes.length > 0) {
    diamondConditions.shape = { in: params.shapes };
  }

  if (params.cuts && params.cuts.length > 0) {
    diamondConditions.cut = { in: params.cuts };
  }

  if (params.caratMin !== undefined || params.caratMax !== undefined) {
    diamondConditions.carat = {};
    if (params.caratMin !== undefined) diamondConditions.carat.gte = params.caratMin;
    if (params.caratMax !== undefined) diamondConditions.carat.lte = params.caratMax;
  }

  // Terapkan kondisi berlian jika ada parameter 4Cs yang aktif
  if (Object.keys(diamondConditions).length > 0) {
    where.diamonds = {
      some: diamondConditions,
    };
  }

  // 3. Bangun Klausa Pengurutan (ORDER BY)
  let orderBy: Prisma.PieceOrderByWithRelationInput = { createdAt: 'desc' };

  switch (params.sortBy) {
    case 'price_asc':
      orderBy = params.currency === 'USD' ? { priceUsdMinor: 'asc' } : { priceIdrMinor: 'asc' };
      break;
    case 'price_desc':
      orderBy = params.currency === 'USD' ? { priceUsdMinor: 'desc' } : { priceIdrMinor: 'desc' };
      break;
    case 'newest':
    default:
      orderBy = { createdAt: 'desc' };
      break;
  }

  // 4. Eksekusi Kueri Bersama Penghitungan Total dalam Transaksi Ringan
  const [totalItems, piecesRaw] = await prisma.$transaction([
    prisma.piece.count({ where }),
    prisma.piece.findMany({
      where,
      orderBy,
      skip,
      take: limit,
      include: {
        images: {
          orderBy: { displayOrder: 'asc' },
          take: 2,
        },
        diamonds: {
          take: 1,
        },
      },
    }),
  ]);

  // 5. Transformasi Data Output Bersih
  const pieces = piecesRaw.map((p) => {
    const primaryImg = p.images.find((img) => img.isPrimary) ?? p.images[0] ?? null;
    const secondaryImg = p.images.find((img) => !img.isPrimary) ?? p.images[1] ?? null;
    const primaryDiamond = p.diamonds[0] ?? null;

    return {
      id: p.id,
      slug: p.slug,
      sku: p.sku,
      name: p.name,
      metal: p.metal,
      priceIdrMinor: p.priceIdrMinor,
      priceUsdMinor: p.priceUsdMinor,
      availability: p.availability,
      primaryImage: primaryImg?.imageUrl ?? null,
      secondaryImage: secondaryImg?.imageUrl ?? null,
      diamond: primaryDiamond
        ? {
            origin: primaryDiamond.origin,
            carat: Number(primaryDiamond.carat),
            shape: primaryDiamond.shape,
            cut: primaryDiamond.cut,
            colorGrade: primaryDiamond.colorGrade,
            clarityGrade: primaryDiamond.clarityGrade,
          }
        : null,
    };
  });

  const totalPages = Math.ceil(totalItems / limit);

  return {
    pieces,
    pagination: {
      page,
      limit,
      totalItems,
      totalPages,
      hasNextPage: page < totalPages,
    },
  };
}
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

```typescript
// ❌ ANTI-PATTERN 1: Menggunakan query string mentah yang rentan SQL Injection
export async function badRawQuery(userInput: string) {
  // BENCANA KEAMANAN: Jangan pernah gabungkan string input ke dalam query SQL!
  return await prisma.$queryRawUnsafe(`SELECT * FROM pieces WHERE name LIKE '%${userInput}%'`);
}

// ❌ ANTI-PATTERN 2: Mengambil seluruh data lalu memfilter di memori JavaScript (In-Memory Filtering)
export async function badMemoryFilter(params: any) {
  // Buruk: Mengambil 10,000 baris dari database ke memori Node.js, membebani RAM dan CPU
  const allPieces = await prisma.piece.findMany();
  return allPieces.filter((p) => p.metal === params.metal).slice(0, 10);
}

// ❌ ANTI-PATTERN 3: Menjalankan kueri beruntun di dalam perulangan (N+1 Query Problem)
export async function badLoopQueries(pieces: any[]) {
  // Buruk: Mengirim 20 query terpisah ke PostgreSQL untuk mengambil gambar satu per satu
  for (const piece of pieces) {
    piece.images = await prisma.pieceImage.findMany({ where: { pieceId: piece.id } });
  }
}
```

---

## 6. Aturan & Invariants (Rules)

1. **Selalu Memanfaatkan Indeks Komposit:** Setiap kondisi `where` yang sering digunakan wajib didukung oleh `@@index` di skema Prisma (misal: `@@index([categoryId, isPublished])`).
2. **Hindari Masalah N+1:** Selalu sertakan relasi anak yang dibutuhkan menggunakan blok `include` berbatas (`take: 2`), bukan perulangan `findMany` terpisah.
3. **Limit Maksimal Penomoran Halaman:** Parameter `limit` wajib dibatasi maksimal 48 item per halaman guna mencegah pembengkakan memori server.
4. **Isolasi Sanitasi Input:** Semua parameter angka desimal (seperti `caratMin`) wajib dikonversi dan divalidasi sebagai angka numerik murni sebelum diteruskan ke query builder.
5. **Klausa `isPublished: true` Wajib pada Kueri Publik:** Kueri publik dilarang menampilkan karya berstatus draft atau tersembunyi.

---

## 7. Pola & Dokumen Terkait

- [PRISMA_MODEL_PATTERN.md](./PRISMA_MODEL_PATTERN.md) — Definisi indeks skema model
- [DIAMOND_SEARCH.md](../FEATURES/DIAMOND_SEARCH.md) — Spesifikasi fungsional filter berlian
- [CATALOG.md](../FEATURES/CATALOG.md) — Spesifikasi fungsional etalase katalog
