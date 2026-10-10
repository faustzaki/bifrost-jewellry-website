# 🗄️ PATTERN — Pendefinisian Model Prisma, Relasi & Indeks (PRISMA_MODEL_PATTERN)

- **Applies To:** Database / ORM / Schema Design
- **Framework:** Prisma ORM, PostgreSQL 16
- **Last Updated:** 2026-10-11

---

## 1. Kapan Digunakan (When to Use)

Gunakan pola ini untuk:
- Setiap penambahan entitas, tabel, kolom, relasi, indeks, atau enum baru dalam berkas skema `prisma/schema.prisma`.
- Menentukan konfigurasi relasi *Foreign Key* dengan integritas referensial yang aman (`onDelete: Restrict` atau `onDelete: Cascade` yang diperhitungkan).
- Mengoptimalkan performa kueri katalog, pencarian berlian 4Cs, dan pelacakan pesanan melalui pembuatan indeks komposit (*composite indexes*).
- Menjalankan migrasi terstruktur (`prisma migrate dev`) dan pengisian data awal (*seeding*) yang bersih.

---

## 2. Kapan TIDAK Boleh Digunakan (When NOT to Use)

Jangan gunakan pola ini untuk:
- Menuliskan data produk palsu atau karangan (*fabricated dummy data*) di dalam `seed.ts` — data awal hanya boleh memuat kategori dasar, FAQ terverifikasi, atau akun admin terisolasi dengan data riil atau marker `<TBD>`.
- Mengubah skema database secara manual melalui query DDL SQL langsung di PostgreSQL tanpa melalui berkas migrasi Prisma resmi.

---

## 3. Struktur Blueprint (Structure)

```
prisma/
├── schema.prisma              # Definisi model, relasi, enum, dan indeks
├── seed.ts                    # Skrip pengisian data benih awal (seeder)
└── migrations/                # Berkas riwayat migrasi SQL yang dihasilkan Prisma
    └── 20261011000000_init/
        └── migration.sql
```

---

## 4. Contoh Implementasi Nyata (Implementation ✅)

Berikut adalah cetak biru model Prisma untuk karya perhiasan (`Piece`), batu berlian (`Diamond`), dan transaksi pesanan (`Order`) dengan konvensi penamaan PostgreSQL yang tepat (`snake_case` di database, `camelCase` di Prisma client), pemetaan enum, dan indeks performa:

```prisma
// prisma/schema.prisma

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

generator client {
  provider = "prisma-client-js"
}

// -------------------------------------------------------------
// ENUMS
// -------------------------------------------------------------

enum UserRole {
  super_admin
  admin
  customer
}

enum MetalType {
  gold_18k
  gold_14k
  platinum
  rose_gold_18k
}

enum PieceAvailability {
  in_stock
  made_to_order
  reserved
  sold_out
}

enum DiamondOrigin {
  natural
  lab_grown
}

enum DiamondCut {
  excellent
  very_good
  good
  fair
}

enum DiamondShape {
  round
  oval
  emerald
  pear
  princess
  marquise
  cushion
}

enum GradingLab {
  GIA
  IGI
}

enum OrderStatus {
  pending_payment
  paid
  processing
  shipped
  completed
  cancelled
  refunded
}

enum PaymentMethod {
  midtrans_snap
  manual_transfer
}

enum PaymentStatus {
  pending
  awaiting_verification
  paid
  failed
  expired
}

// -------------------------------------------------------------
// MODELS
// -------------------------------------------------------------

model Piece {
  id              String            @id @default(uuid()) @db.Uuid
  slug            String            @unique @db.VarChar(140)
  sku             String            @unique @db.VarChar(60)
  name            String            @db.VarChar(200)
  description     String            @db.Text
  categoryId      String            @map("category_id") @db.Uuid
  collectionId    String?           @map("collection_id") @db.Uuid
  metal           MetalType
  settingType     String            @map("setting_type") @db.VarChar(100)
  priceIdrMinor   BigInt            @map("price_idr_minor") // Menyimpan nilai minor IDR (misal: Rp 45.000.000 -> 4500000000)
  priceUsdMinor   Int               @map("price_usd_minor") // Menyimpan nilai minor USD sen (misal: $3,200.00 -> 320000)
  availability    PieceAvailability @default(in_stock)
  isPublished     Boolean           @default(false) @map("is_published")
  isFeatured      Boolean           @default(false) @map("is_featured")
  createdAt       DateTime          @default(now()) @map("created_at") @db.Timestamptz(6)
  updatedAt       DateTime          @updatedAt @map("updated_at") @db.Timestamptz(6)

  // Relasi
  diamonds        Diamond[]
  variants        PieceVariant[]
  images          PieceImage[]
  orderLines      OrderLine[]

  // Indeks Performa untuk Kueri Katalog
  @@index([categoryId, isPublished])
  @@index([collectionId, isPublished])
  @@index([availability, isPublished])
  @@map("pieces")
}

model Diamond {
  id                String        @id @default(uuid()) @db.Uuid
  pieceId           String        @map("piece_id") @db.Uuid
  origin            DiamondOrigin
  carat             Decimal       @db.Decimal(5, 2)
  cut               DiamondCut
  colorGrade        String        @map("color_grade") @db.VarChar(10)
  clarityGrade      String        @map("clarity_grade") @db.VarChar(10)
  shape             DiamondShape
  gradingLab        GradingLab    @map("grading_lab")
  certificateNumber String        @unique @map("certificate_number") @db.VarChar(60)
  certificateUrl    String?       @map("certificate_url") @db.Text
  createdAt         DateTime      @default(now()) @map("created_at") @db.Timestamptz(6)

  // Relasi ke Piece
  piece             Piece         @relation(fields: [pieceId], references: [id], onDelete: Cascade)

  // Indeks untuk Pencarian Berlian 4Cs
  @@index([origin, carat])
  @@index([shape, origin])
  @@index([gradingLab, certificateNumber])
  @@map("diamonds")
}

model PieceVariant {
  id        String   @id @default(uuid()) @db.Uuid
  pieceId   String   @map("piece_id") @db.Uuid
  ringSize  String?  @map("ring_size") @db.VarChar(20)
  stock     Int      @default(0)
  skuSuffix String   @map("sku_suffix") @db.VarChar(30)
  createdAt DateTime @default(now()) @map("created_at") @db.Timestamptz(6)

  piece      Piece       @relation(fields: [pieceId], references: [id], onDelete: Cascade)
  orderLines OrderLine[]

  @@unique([pieceId, ringSize])
  @@map("piece_variants")
}

model PieceImage {
  id           String   @id @default(uuid()) @db.Uuid
  pieceId      String   @map("piece_id") @db.Uuid
  imageUrl     String   @map("image_url") @db.Text
  altText      String   @map("alt_text") @db.VarChar(200)
  displayOrder Int      @default(0) @map("display_order")
  isPrimary    Boolean  @default(false) @map("is_primary")
  createdAt    DateTime @default(now()) @map("created_at") @db.Timestamptz(6)

  piece        Piece    @relation(fields: [pieceId], references: [id], onDelete: Cascade)

  @@index([pieceId, displayOrder])
  @@map("piece_images")
}
```

### Skrip Seeding yang Aman (`prisma/seed.ts`)

```typescript
// prisma/seed.ts
import { PrismaClient, UserRole } from '@prisma/client';
import bcrypt from 'bcryptjs';

const prisma = new PrismaClient();

async function main() {
  console.log('Memulai inisialisasi basis data (Seeding)...');

  // 1. Seed Akun Admin dari variabel lingkungan (jika dikonfigurasi)
  const adminEmail = process.env.SEED_ADMIN_EMAIL;
  const adminPassword = process.env.SEED_ADMIN_PASSWORD;

  if (adminEmail && adminPassword) {
    const existingAdmin = await prisma.user.findUnique({
      where: { email: adminEmail },
    });

    if (!existingAdmin) {
      const hashedPassword = await bcrypt.hash(adminPassword, 12);
      await prisma.user.create({
        data: {
          email: adminEmail,
          passwordHash: hashedPassword,
          fullName: 'Bifrost Master Administrator',
          role: UserRole.super_admin,
          isActive: true,
        },
      });
      console.log(`[Seed] Super admin (${adminEmail}) berhasil dibuat.`);
    }
  }

  // 2. Data Butik Fisik Utama (menggunakan marker TBD jika belum final)
  const flagshipBoutique = await prisma.boutique.upsert({
    where: { slug: 'flagship-boutique-jakarta' },
    update: {},
    create: {
      name: 'Bifrost Flagship Boutique Jakarta',
      slug: 'flagship-boutique-jakarta',
      addressLine: '<TBD: alamat fisik butik Bifrost Jewellery>',
      city: 'Jakarta Selatan',
      provinceOrState: 'DKI Jakarta',
      postalCode: '12190',
      phone: '<TBD: nomor telepon resmi butik>',
      mapUrl: '<TBD: link Google Maps butik>',
      openingHours: '<TBD: jam buka operasional butik>',
      isActive: true,
      displayOrder: 1,
    },
  });
  console.log(`[Seed] Butik fisik: ${flagshipBoutique.name} siap.`);
}

main()
  .catch((e) => {
    console.error('Kegagalan saat seeding:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

---

## 5. Anti-Pattern (Anti-Pattern ❌)

```prisma
// ❌ ANTI-PATTERN 1: Menggunakan Float untuk harga (mengakibatkan pembulatan biner tidak akurat)
model BadPiece {
  price Float // DILARANG: Terjadi bug presisi desimal floating point
}

// ❌ ANTI-PATTERN 2: Mengabaikan pemetaan snake_case database
model BadUser {
  id String @id
  fullName String // Di PostgreSQL menjadi "fullName" (perlu kutip dua) bukan full_name
}

// ❌ ANTI-PATTERN 3: Menghubungkan relasi penting dengan onDelete: Cascade yang berbahaya
model BadOrder {
  userId String
  // Jika akun user dihapus, data faktur pesanan keuangan ikut terhapus!
  user User @relation(fields: [userId], references: [id], onDelete: Cascade) // Seharusnya onDelete: Restrict / SetNull
}
```

---

## 6. Aturan & Invariants (Rules)

1. **Pemetaan Penamaan Eksplisit:** Nama tabel di basis data wajib jamak `snake_case` melalui `@@map("table_names")`, dan setiap kolom wajib menggunakan `@map("column_name")`.
2. **Uang Selalu Minor Integer:** Dilarang menggunakan tipe `Float` untuk nilai uang. Gunakan `BigInt` untuk IDR dan `Int` untuk USD sen guna menjamin presisi mutlak.
3. **Indeks Wajib pada Kolom Filter Sering:** Setiap kombinasi filter pencarian yang sering diakses (seperti `[categoryId, isPublished]` dan `[origin, carat]`) wajib memiliki `@@index`.
4. **Relasi Unik Terjaga:** Sertifikat berlian (`certificateNumber`), nomor pesanan (`orderNumber`), dan SKU (`sku`) wajib memiliki batasan `@unique`.
5. **Prisma Migrations Resmi:** Seluruh perubahan skema wajib dieksekusi dengan perintah `npx prisma migrate dev --name <deskripsi_jelas>`.

---

## 7. Pola & Dokumen Terkait

- [SERVER_ACTION_PATTERN.md](./SERVER_ACTION_PATTERN.md) — Konsumsi model Prisma di Server Actions
- [CATALOG_QUERY_PATTERN.md](./CATALOG_QUERY_PATTERN.md) — Kueri terindeks katalog
- [DATABASE.md](../DATABASE.md) — Referensi skema lengkap sistem
