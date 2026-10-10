# Bifrost Jewellery — `.AGENTS/` AI Documentation Folder

## Context

The user wants a project folder containing the `.md` documentation set that an AI agent reads before working on the project: everything about the product, systems, architecture, and design. The project is **Bifrost Jewellery** — an online diamond/jewellery shopping website with a **quiet-luxury** theme. Target directory `C:\Belajar Coding\Projects\bifrost-jewellery-website\` currently exists and is **empty**.

Deliverable: a complete, self-contained `.AGENTS/` documentation folder in that directory, matching the structure and depth of the workspace's existing master template and its filled sibling copies, but populated with this project's real decisions so a fresh AI agent can work with zero re-explanation. **No runtime/business code is created in this task** — scope is the `.md` documentation folder only. Nothing outside `bifrost-jewellery-website/` is created or modified (in particular: the master template at `Projects/.agents/` and any omp root config are untouched — read-only references).

### Locked decisions (from user, this session — do not revisit)

| Topic | Decision |
|---|---|
| Doc folder name | `.AGENTS/` (upper) |
| Stack | Next.js 15 + TypeScript + PostgreSQL |
| Commerce model | **Hybrid** — cart + checkout (pay online) OR request consultation |
| Diamond offering | **Both** natural + lab-grown |
| Boutique | Physical boutique exists **+ online appointment booking** |
| Brand name | **Bifrost Jewellery** |
| Customer accounts | **Guest checkout + optional account** (register post-purchase OK) |
| Admin | **In-app admin panel inside the Next.js app** (auth + role) |
| Language | **English only** — public copy AND documents |
| Market / payment | **IDR + USD display; Midtrans Snap + manual bank transfer option** |

### Conventions to copy (verified this session)

- Source of truth for structure: `C:\Belajar Coding\Projects\.agents\` — `AGENTS.md` (router), Tier 1/2/3 files, `MANUAL.md` (filling guide), `DECISIONS/_TEMPLATE.md`, `PATTERNS/_TEMPLATE.md`, `FEATURES/README.md` pattern.
- Depth reference (Next.js): `C:\Belajar Coding\Projects\bifrost-website\.AGENTS\` — filled `PROJECT_CONTEXT.md`, `CODING_RULES.md`, `STACK.md`, `ARCHITECTURE.md`, `DATABASE.md`, `API_CONTRACT.md`, `UI_UX.md`, `GLOSSARY.md`.
- Depth reference (Laravel, but has the best Tier-0 + ADR + FEATURES index patterns): `C:\Belajar Coding\Projects\PT Swastika Berdikari Website\.agents\` — `CURRENT_STATE.md`, `FEATURES/README.md`, `DECISIONS/001-*.md`, `PATTERNS/SERVICE_PATTERN.md`.
- Reuse conventions verbatim where they are brand-neutral: the `AGENTS.md` global-constraints list and routing-tree shape, the `_TEMPLATE.md` files, the `FEATURES/README.md` feature-map table, `TASKS/BUGS.md` bug-report template, `PROMPTS/*` prompt templates.

---

## Approach

Write files in the order below (**foundation first so later files can reference settled facts**). The doc language is **English** throughout; headings follow the master template's numbered style. Each doc must be internally consistent with the decisions table above — **no fabricated business facts**: any value not derivable (address, phone, domain, real product data) is written as an explicit `<TBD: …>` marker and collected in `CURRENT_STATE.md §Open Inputs` (see Step 15), never invented.

### Directory layout to create

```
bifrost-jewellery-website/
└── .AGENTS/
    ├── AGENTS.md
    ├── PROJECT_CONTEXT.md
    ├── CODING_RULES.md
    ├── GLOSSARY.md
    ├── CURRENT_STATE.md
    ├── ARCHITECTURE.md
    ├── STACK.md
    ├── DATABASE.md
    ├── API_CONTRACT.md
    ├── UI_UX.md
    ├── MANUAL.md
    ├── DECISIONS/
    │   ├── README.md
    │   ├── _TEMPLATE.md
    │   ├── 001-nextjs15-app-router-postgres.md
    │   ├── 002-quiet-luxury-design-direction.md
    │   ├── 003-hybrid-commerce-model.md
    │   ├── 004-guest-checkout-optional-account.md
    │   ├── 005-dual-origin-natural-lab-grown.md
    │   ├── 006-midtrans-snap-payments.md
    │   ├── 007-in-app-admin-panel.md
    │   └── 008-english-only-content.md
    ├── FEATURES/
    │   ├── README.md
    │   ├── CATALOG.md
    │   ├── DIAMOND_SEARCH.md
    │   ├── PRODUCT_DETAIL.md
    │   ├── CART_CHECKOUT.md
    │   ├── PAYMENT.md
    │   ├── ORDERS.md
    │   ├── ACCOUNT.md
    │   ├── APPOINTMENT.md
    │   ├── BOUTIQUE.md
    │   ├── CONSULTATION.md
    │   ├── EDITORIAL.md
    │   ├── AUTH.md
    │   ├── ADMIN.md
    │   └── LEGAL_AND_FAQ.md
    ├── FLOWS/
    │   ├── README.md
    │   ├── BROWSING_FLOW.md
    │   ├── CHECKOUT_FLOW.md
    │   ├── APPOINTMENT_FLOW.md
    │   └── ADMIN_FLOW.md
    ├── PATTERNS/
    │   ├── README.md
    │   ├── _TEMPLATE.md
    │   ├── SERVER_ACTION_PATTERN.md
    │   ├── PRISMA_MODEL_PATTERN.md
    │   ├── COMPONENT_PATTERN.md
    │   ├── API_RESPONSE_PATTERN.md
    │   ├── CATALOG_QUERY_PATTERN.md
    │   └── PAYMENT_WEBHOOK_PATTERN.md
    ├── TASKS/
    │   ├── TODO.md
    │   ├── ROADMAP.md
    │   └── BUGS.md
    └── PROMPTS/
        ├── UI_PROMPT.md
        ├── BACKEND_PROMPT.md
        └── REFACTOR_PROMPT.md
```

**File count: 55 total** — `.AGENTS/` 11 · `DECISIONS/` 10 · `FEATURES/` 15 · `FLOWS/` 5 · `PATTERNS/` 8 · `TASKS/` 3 · `PROMPTS/` 3.

### Step 1 — `PROJECT_CONTEXT.md` (write first: everything else references it)

Adapt the sibling `PROJECT_CONTEXT.md` section skeleton (Identity → Description → Target Users → Business Requirements → Scope → Stakeholders → Success Metrics → Timeline → Related Documents). Fill with:

- Identity: Project Name `Bifrost Jewellery`, Project Code `bifrost-jewellery-website`, Version `v0.1.0 (Planning)`, Status `Planning`, Repository URL `<TBD: git remote>`, Domain `<TBD: domain>`, Last Updated `2026-10-11`, Design Direction `Quiet Luxury Editorial`.
- Description: online boutique for fine diamond jewellery; both natural and lab-grown; purchase online or book a private consultation/appointment; positioning = restraint, provenance, certification transparency.
- Target Users table rows: **Visitors/Shoppers** (research → purchase), **Account holders** (order history, wishlist), **Admin/Ops** (catalog, orders, appointments, content), **Boutique staff** (appointment fulfilment).
- Functional Requirements — assign these **exact IDs and one-line descriptions** (each becomes the FR referenced by the matching `FEATURES/` doc):
  - `FR-001` Catalog & collection browsing
  - `FR-002` Diamond search & filter (4Cs, origin, price, shape)
  - `FR-003` Product detail with certification, imagery, size guide
  - `FR-004` Cart with guest + account support
  - `FR-005` Hybrid checkout — pay online (Midtrans Snap) OR request consultation
  - `FR-006` Manual bank-transfer payment path
  - `FR-007` Order status tracking
  - `FR-008` Optional customer account (wishlist, order history, addresses)
  - `FR-009` Boutique appointment booking
  - `FR-010` Boutique locator
  - `FR-011` Consultation / inquiry form (hybrid path)
  - `FR-012` Editorial pages (journal, about, size & care guides)
  - `FR-013` Admin panel (products, orders, appointments, content, users)
  - `FR-014` Auth & role-based access control
  - `FR-015` SEO + structured data (`Product`, `Offer`, `LocalBusiness`)
  - `FR-016` Site search
- Non-Functional Requirements — `NFR-001` Lighthouse Performance ≥ 90 & SEO = 100; `NFR-002` Accessibility ≥ 90, WCAG AA contrast, `prefers-reduced-motion` honoured; `NFR-003` Core Web Vitals LCP < 2.5s / CLS < 0.1 / INP < 200ms; `NFR-004` all money in minor units (IDR/USD) with server-side revalidation at checkout; `NFR-005` PCI scope kept out of app (Midtrans Snap hosted page).
- Scope — In Scope: the FR list above. Out of Scope (one inline line each, no section): native mobile app; multi-vendor marketplace; AR/virtual try-on; AI chat; auctions/second-hand trade-in; currencies beyond IDR/USD.
- Success Metrics, Stakeholders, Timeline table (Phase 1 Documentation → Phase 2 Scaffold → Phase 3 Catalog → Phase 4 Commerce → Phase 5 Appointment/Admin → Phase 6 Launch).

### Step 2 — `STACK.md`

Adapt the sibling `STACK.md` skeleton. Fill exact tables:

- Overview: Frontend `Next.js 15 (App Router)`; Backend `Next.js Route Handlers + Server Actions`; Database `PostgreSQL 16`; Language `TypeScript 5.x (strict)`.
- Frontend/App packages table with Function column: `next` (App Router), `react` / `react-dom` 19, `typescript`, `tailwindcss` v4, `zod`, `react-hook-form` + `@hookform/resolvers`, `framer-motion`, `lenis`, `@phosphor-icons/react`, `next-auth` (Auth.js v5), `@prisma/client`, `midtrans-client`, `resend`, `sharp`.
- Data layer: `prisma` (migrations/seed), `@vercel/blob` (image storage). State: server components + server actions; no global client store.
- DevOps: npm, Vercel deploy, ESLint + Prettier, Vitest (unit) + Playwright (E2E).
- Third-party services: Midtrans (payments), Resend (transactional email), Vercel Blob (images), Vercel Analytics. Payment gateway row explicitly records the **bank-transfer manual verification** as an operational path (not a vendor).
- Fonts table: `Cormorant Garamond` (display serif), `Inter` (body/UI), `JetBrains Mono` (certificate numbers / order & SKU codes).
- Environment variables block (names only, no secrets): `DATABASE_URL`, `DIRECT_URL`, `AUTH_SECRET`, `AUTH_URL`, `NEXT_PUBLIC_SITE_URL`, `MIDTRANS_SERVER_KEY`, `NEXT_PUBLIC_MIDTRANS_CLIENT_KEY`, `MIDTRANS_IS_PRODUCTION`, `RESEND_API_KEY`, `EMAIL_FROM`, `BLOB_READ_WRITE_TOKEN`.
- Compatibility: Node ≥ 20, modern browsers, mobile-first.

### Step 3 — `GLOSSARY.md`

Adapt the master `GLOSSARY.md` skeleton (Core Entities / Status Definitions / Roles / Business Terms / Technical Terms / Abbreviations). Fill:

- Core Entities (each with a `_Do not use:_` alias line): **Piece** (a jewellery product; do not use "item"/"product" interchangeably in copy), **Diamond** (the stone; `origin` = `natural` | `lab_grown`), **4Cs** (Cut, Color, Clarity, Carat — always capitalised), **Certificate** (third-party grading report; `grading_lab` = `GIA` | `IGI`, `certificate_number`), **Setting** (metalwork; `metal` enum), **Collection** (curated grouping), **Boutique** (physical location), **Appointment** (booked in-person visit), **Inquiry** (consultation request), **Cart**, **Order**, **Order Line**, **Wishlist**.
- Status Definitions — exact enum values used across DB/UI:
  - Order: `pending_payment` → `paid` → `processing` → `shipped` → `completed`; terminal `cancelled`, `refunded`.
  - Payment: `pending` → `paid` | `failed` | `expired`; manual transfer uses `awaiting_verification` between `pending` and `paid`. Payment methods: `midtrans_snap`, `manual_transfer`.
  - Appointment: `requested` → `confirmed` → `completed`; terminal `cancelled`, `no_show`.
  - Piece availability: `in_stock`, `made_to_order`, `reserved`, `sold_out`.
  - Diamond origin: `natural`, `lab_grown`.
- Roles: `super_admin`, `admin`, `customer`, `guest`.
- Business Terms: One-of-a-kind, Made-to-order, Provenance, Consignment, Ring size, Complimentary resizing, Insurance-grade appraisal, Certificate match.
- Abbreviations: GIA, IGI, 4Cs, TBD-as-`<TBD>`, SKU, RBAC, CTA, ISR, SSG, SEO, JSON-LD, VAT/PPN, IDR, USD.

### Step 4 — `CODING_RULES.md`

Adapt the sibling (Next.js) `CODING_RULES.md` skeleton — it is stack-correct. Fill:

- General: Server Components by default; `"use client"` only for hooks/browser APIs; DRY util in `src/lib/`; KISS; SRP.
- File/folder conventions table: components `src/components/` kebab-case `.tsx`; pages `src/app/**/page.tsx`; server actions `src/app/**/actions.ts` or `src/lib/actions/`; data access `src/lib/db/`; validation schemas `src/lib/validation/` (Zod); types `src/lib/types/`; Prisma `prisma/schema.prisma`, `prisma/seed.ts`.
- Next.js/React rules: page files are Server Components; `next/link`, `next/image` (explicit dimensions or `fill` + `sizes`); `generateMetadata()` per page; **prices recomputed server-side** — never trust client-sent amounts; server actions must re-validate with Zod.
- Styling: Tailwind v4 semantic tokens only (table reproduced from `UI_UX.md §2`); no arbitrary hex in components; no inline `style` except dynamic values; mobile-first.
- TypeScript: **no `any`**; every domain shape typed in `src/lib/types/`; `async/await` with `try/catch` on all I/O; no `node:*` imports in client components.
- Money: integer minor units in DB (`price_idr_minor`, `price_usd_minor`); formatting only via `src/lib/money.ts`.
- a11y, performance, git workflow (Conventional Commits, English messages), testing standards (Vitest ≥ 80% on `src/lib/`, Playwright happy paths).

### Step 5 — `UI_UX.md` (design system — the quiet-luxury source of truth)

Adapt the sibling UI_UX skeleton but this file is fully bespoke. Must contain:

- **§1 Design principles:** restraint; whitespace as structure; one hero image per view; no discount/loud retail language; jewellery photography carries the page; motion is quiet.
- **§2 Color tokens** — exact values (contrast-verified, WCAG AA):

  | Token | Value | Use | Contrast check |
  |---|---|---|---|
  | `--ql-canvas` | `#FAF8F5` | page background (warm off-white) | — |
  | `--ql-surface` | `#FFFFFF` | cards, sheets | — |
  | `--ql-ink` | `#1A1A1A` | primary text | 16.42:1 on canvas ✅ AA/AAA |
  | `--ql-ink-muted` | `#6B6259` | secondary text | 5.63:1 on canvas ✅ AA |
  | `--ql-line` | `#E7E2DA` | hairlines, borders | non-text |
  | `--ql-gold` | `#9C7C3F` | ornament / display-size accent only | 3.69:1 on canvas — **≥24px only**; 7.26:1 on ink ✅ |
  | `--ql-gold-deep` | `#7C6236` | gold **small text** on canvas | 5.42:1 ✅ AA |
  | `--ql-ink-surface` | `#1A1A1A` | dark editorial blocks / footer | white text 16.42:1 ✅ |
  | `--ql-success` | `#15803D` | success | — |
  | `--ql-error` | `#B3261E` | error | — |

  Rule lines: gold NEVER as normal-size text on canvas (use `--ql-gold-deep`); no pure black `#000`; no gradients as decoration.
- **§3 Typography:** `Cormorant Garamond` display (weights 400/500), `Inter` body/UI, `JetBrains Mono` for certificate/SKU/order codes. Locked type scale table with steps `xs 12 → sm 14 → base 16 → md 18 → lg 24 → xl 32 → 2xl 48 → 3xl 64 → 4xl 88` (rem values, usage column) — forbid sizes off-list.
- **§4 Spacing & layout:** container max-width 1280px, side padding 20px mobile / 48px desktop; 12-col desktop grid; section rhythm `py-24 lg:py-32`; radius: cards 2px (near-square, editorial), buttons pill; hairline borders only, no heavy shadows.
- **§5 Components table** (Component | Variants): Button (`primary`, `outline`, `quiet`), Input, Select, QuantityStepper, ProductCard, CollectionCard, FilterPanel, Gallery/Lightbox, SizeGuide, Badge (`origin`, `availability`, `certification`), Modal, Toast, Accordion (FAQ), Navbar (`transparent-over-hero`, `solid`), Footer, AppointmentPicker, Tabs.
- **§6 Motion:** fade/opacity + slight translate, 300–500ms, ease `cubic-bezier(0.22,1,0.36,1)`; Lenis smooth scroll; honour `prefers-reduced-motion`; forbidden: parallax, scroll-hijack, marquee, auto-rotating carousels that steal focus.
- **§7 Accessibility checklist** (the master `UI_UX.md` checklist verbatim, threshold 4.5:1 / 3:1).
- **§8 Content & voice (quiet-luxury copy rules) — hard rules:** no exclamation marks; no "Sale/Discount/% off/Deal"; no ALL-CAPS shouting (section labels may be uppercase 12px/0.12em tracking); sentence case for headings; describe provenance/craft not urgency; never invent certification claims; every claim about a stone must trace to a `certificate_number`.
- **§9 Forbidden (hard rules):** pure black; gold as body text; more than one accent per view; stock-photo jewellery; fake testimonials; countdown timers; pop-ups on entry.

### Step 6 — `ARCHITECTURE.md`

Adapt sibling skeleton. Fill:

- Overview: pattern `Modular monolith (Next.js full-stack)`; communication `Server Components + Server Actions + Route Handlers (REST for webhooks/public GET)`; auth `Auth.js v5 (credentials + role claims)`; deployment `Vercel + managed Postgres`.
- High-level mermaid `graph TB` — Client → Next.js (RSC, Server Actions, Route Handlers, Prisma client) → PostgreSQL; side nodes: Midtrans (Snap token + webhook), Resend (email), Vercel Blob (images).
- Component breakdown: App shell (`src/app/layout.tsx`), catalog rendering (SSG/ISR for `/`, `/collections/**`, `/pieces/**`; dynamic for `/account`, `/admin`, `/checkout`), data access (`src/lib/db/**` Prisma wrappers), auth (`src/lib/auth.ts`), payments (`src/lib/payments/midtrans.ts`), email (`src/lib/email/**`).
- Data flow mermaid `sequenceDiagram`: shopper → cart → `POST /api/checkout` (server recomputes totals, creates Order `pending_payment`, requests Snap token) → Snap hosted page → Midtrans webhook → `POST /api/payments/midtrans/webhook` (verify signature, set Payment `paid`, Order `paid`, send email).
- Folder structure block for `src/` and `prisma/`.
- Deployment mermaid + environments (local `http://localhost:3000`, preview on Vercel, production `<TBD: domain>`).
- Security section: RBAC on admin routes (server-side session check, not middleware-only), Zod validation on every action/handler, Midtrans webhook signature verification + idempotency key, rate limiting on auth/appointment/inquiry endpoints, secrets in env only, no card data touching app.

### Step 7 — `DATABASE.md`

Adapt master `DATABASE.md` skeleton. Fill:

- Info: Engine `PostgreSQL 16`, Charset `UTF8`, ORM `Prisma`, migrations `prisma migrate`.
- Naming table (same as master: snake_case plural tables, `{singular}_id` FKs, `is_`/`has_` booleans, `*_at` timestamps).
- ERD mermaid `erDiagram` covering: `users`, `addresses`, `categories`, `collections`, `pieces`, `piece_images`, `piece_variants`, `diamonds`, `carts`, `cart_lines`, `orders`, `order_lines`, `payments`, `appointments`, `boutiques`, `inquiries`, `testimonials`, `faqs`, `wishlist_items`.
- §4 Table Schemas — one subsection per table with a Column/Type/Constraints/Description table. Minimum required columns to pin (implementer adds obvious timestamps `created_at`/`updated_at`):
  - `users`: `id` uuid PK, `email` citext unique, `password_hash` nullable (invited admins), `full_name`, `phone` nullable, `role` enum(`super_admin`,`admin`,`customer`) default `customer`, `is_active` bool, `email_verified_at` nullable.
  - `pieces`: `id` uuid, `slug` unique, `sku` unique, `name`, `description`, `category_id` FK, `collection_id` FK nullable, `metal` enum(`gold_18k`,`gold_14k`,`platinum`,`rose_gold_18k`), `setting_type` text, `price_idr_minor` int, `price_usd_minor` int, `availability` enum(`in_stock`,`made_to_order`,`reserved`,`sold_out`), `is_published` bool, `is_featured` bool.
  - `diamonds`: `id` uuid, `piece_id` FK, `origin` enum(`natural`,`lab_grown`), `carat` numeric(5,2), `cut` enum(`excellent`,`very_good`,`good`,`fair`), `color_grade` text, `clarity_grade` text, `shape` enum(`round`,`oval`,`emerald`,`pear`,`princess`,`marquise`,`cushion`), `grading_lab` enum(`GIA`,`IGI`), `certificate_number` text, `certificate_url` text nullable.
  - `piece_variants`: `id` uuid, `piece_id` FK, `ring_size` text nullable, `stock` int, `sku_suffix` text.
  - `cart_lines` / `order_lines`: `id`, `piece_id` FK, `variant_id` FK nullable, `quantity` int, `unit_price_minor_idr` int, `unit_price_minor_usd` int (order lines snapshot price; cart lines reference current).
  - `orders`: `id` uuid, `order_number` text unique, `user_id` FK nullable (**guest allowed**), `guest_email` nullable, `status` enum (Glossary values), `currency` enum(`IDR`,`USD`), `subtotal_minor` int, `shipping_minor` int, `total_minor` int, `shipping_address_id` FK nullable, `notes` nullable.
  - `payments`: `id` uuid, `order_id` FK, `method` enum(`midtrans_snap`,`manual_transfer`), `status` enum(`pending`,`awaiting_verification`,`paid`,`failed`,`expired`), `provider_ref` text nullable (Midtrans transaction id), `amount_minor` int, `verified_by` FK users nullable, `verified_at` nullable.
  - `appointments`: `id` uuid, `user_id` nullable, `guest_name`, `guest_email`, `guest_phone`, `boutique_id` FK, `scheduled_at` timestamptz, `status` enum, `piece_id` FK nullable (what they want to see), `notes`.
  - `boutiques`: `id`, `name`, `address_line`, `city`, `map_url`, `phone`, `opening_hours`, `is_active`.
  - `inquiries`: `id`, `user_id` nullable, `name`, `email`, `phone`, `piece_id` nullable, `message`, `status` enum(`new`,`contacted`,`closed`).
- §5 Indexes table (e.g. `pieces.slug` unique, `pieces.category_id`, `diamonds.certificate_number` unique, `orders.order_number` unique, `appointments.scheduled_at`).
- §6 Migration rules (Prisma migrations only; no manual DDL; descriptive names).
- §7 Seeder strategy: `prisma/seed.ts` seeds `boutiques`, `categories`, `faqs` (real content once supplied, else `<TBD>` markers, **never invented product rows**); admin account seeded from env `SEED_ADMIN_EMAIL` in dev only.

### Step 8 — `API_CONTRACT.md`

Adapt master skeleton. Fill:

- Config: Base URL dev `http://localhost:3000`, prod `<TBD: domain>`; no version prefix; Content-Type `application/json`; auth `Auth.js session cookie` (admin) / none (public GET).
- Standard response format blocks (success / error) copied from master.
- Endpoints — enumerate exactly these:
  - `GET /api/pieces` — list + filter query (`origin`, `shape`, `carat_min`, `carat_max`, `price_max`, `category`, `collection`, `availability`, `sort`, `page`); public; `200`/`400`.
  - `GET /api/pieces/[slug]` — single piece w/ diamonds, images, variants; public; `200`/`404`.
  - `POST /api/checkout` — body `{ cartLines[], currency, guestEmail?, shippingAddress }` → server recomputes prices, creates order, returns `{ orderNumber, snapToken }`; public; `201`/`400`/`409`.
  - `POST /api/payments/midtrans/webhook` — Midtrans notification; verifies signature; idempotent; `200`/`401`/`409`.
  - `POST /api/payments/manual-transfer` — submits transfer proof reference → sets `awaiting_verification`; `201`/`400`.
  - `POST /api/appointments` — `{ boutiqueId, scheduledAt, guest*, pieceId? }` → `201`/`400`/`409` (slot taken).
  - `GET /api/appointments/slots?boutiqueId=&date=` — public availability; `200`.
  - `POST /api/inquiries` — consultation request; `201`/`400`.
  - `POST /api/auth/[...nextauth]` — Auth.js handler.
  - Admin mutations are **Server Actions, not REST** — state this explicitly and point to `PATTERNS/SERVER_ACTION_PATTERN.md`.
- HTTP status code table (master's).
- Rate limiting: auth `5/min/IP`, appointments `10/min/IP`, inquiries `5/min/IP`, public GET `120/min/IP`.

### Step 9 — `CURRENT_STATE.md` (Tier 0 — session memory)

Adapt the PT Swastika `CURRENT_STATE.md` structure (Rules for AI → Snapshot → Last Session → Next Steps). Content: snapshot date `2026-10-11`, status `Planning — documentation complete, no code yet`, branch `main`. Must end with an **§Open Inputs** list of every `<TBD>` used across the docs (domain, git remote, boutique address/phone/hours/maps, business WhatsApp/contact email, legal entity for legal pages, product catalog data, price bands, Midtrans merchant keys, which grading labs are accepted as suppliers). Rule: agent updates this file at the end of every session.

### Step 10 — `AGENTS.md` (router — write after the above so the index is accurate)

Adapt the master `AGENTS.md` + PT Swastika variant. Fill:

- Global Constraints: reuse the master's 10, plus: (11) admin mutations are Server Actions re-validated with Zod; (12) never trust client-submitted prices — recompute server-side; (13) language of docs and copy is English (ADR-008).
- Tier 0 table: `CURRENT_STATE.md` (always first read, last updated).
- Tier 1: `AGENTS.md`, `PROJECT_CONTEXT.md`, `CODING_RULES.md`, `GLOSSARY.md`.
- Tier 2: `ARCHITECTURE.md`, `STACK.md`, `DATABASE.md`, `API_CONTRACT.md`, `UI_UX.md`.
- Tier 3 tables listing the **actual** files from the Directory layout above (FEATURES names, FLOWS names, DECISIONS names, PATTERNS names, TASKS, PROMPTS).
- Routing tree (mirror master's, adapted to this stack; "frontend" → UI_UX + COMPONENT_PATTERN; "backend/API" → API_CONTRACT + DATABASE + SERVER_ACTION_PATTERN; "database" → DATABASE + PRISMA_MODEL_PATTERN; "payments" → FEATURES/PAYMENT + PATTERNS/PAYMENT_WEBHOOK_PATTERN).
- Update Protocol (master's 9 rules + "update CURRENT_STATE every session").
- **"First-time fill order" note** (answers "where do we start"): `PROJECT_CONTEXT → STACK → GLOSSARY → CODING_RULES → ARCHITECTURE → DATABASE → API_CONTRACT → UI_UX → FEATURES/* → FLOWS/* → DECISIONS/* → TASKS/*`.
- Related Documents footer.

### Step 11 — `MANUAL.md`

Copy the master `MANUAL.md`, then adjust: rename `.AI/` references to `.AGENTS/`; replace the example feature file list with this project's `FEATURES/` names; keep the filling-guide, routing explanation, best-practices DO/DON'T tables, and FAQ (which are brand-neutral). Add one FAQ entry: "Docs are English-only by decision (ADR-008)".

### Step 12 — `DECISIONS/` (8 ADRs + README + template)

- `_TEMPLATE.md`: copy verbatim from `Projects/.agents/DECISIONS/_TEMPLATE.md`.
- `README.md`: index table (Status | Date) listing all 8 with status `Accepted`, date `2026-10-11`.
- Each ADR: use the template's sections (Status/Date/Deciders/Related ADR → Context → Decision → Alternatives Considered → Consequences → References). One ADR per locked decision:
  - `001-nextjs15-app-router-postgres.md` (alternatives: Laravel monolith, decoupled Next+Laravel, Astro+headless).
  - `002-quiet-luxury-design-direction.md` (alternatives: bold retail e-commerce, dark glamour, minimalist-SaaS).
  - `003-hybrid-commerce-model.md` (alternatives: inquiry-only, full self-serve checkout).
  - `004-guest-checkout-optional-account.md` (alternatives: mandatory account, guest-only).
  - `005-dual-origin-natural-lab-grown.md` (alternatives: natural-only, lab-only).
  - `006-midtrans-snap-payments.md` (alternatives: Stripe, Xendit, no online payment).
  - `007-in-app-admin-panel.md` (alternatives: headless CMS, no admin).
  - `008-english-only-content.md` (alternatives: bilingual ID+EN, Indonesian-only).

### Step 13 — `FEATURES/` (README + 15 specs)

- `README.md`: feature-map table (`Feature | File | Scope | FR | Status` = `Planned`) copied in shape from PT Swastika's `FEATURES/README.md`, plus an Out-of-Scope list matching `PROJECT_CONTEXT` §Scope.
- Each spec file: copy PT Swastika's per-feature section shape (Status / Scope / Primary Goal → Overview → User Stories → Business Rules → Data (`DATABASE.md` tables) → Endpoints (`API_CONTRACT.md`) → UI (`UI_UX.md` components) → Acceptance Criteria → Related Documents). Every acceptance criterion must be observable. Key specifics to pin:
  - `CATALOG.md`: collections grid, filtering entry, SSR/ISR caching.
  - `DIAMOND_SEARCH.md`: filter fields = origin, shape, carat range, colour, clarity, price range, availability; sorting; empty-result state.
  - `PRODUCT_DETAIL.md`: certificate block (lab + number + link), 4Cs table, size guide, add-to-cart, "Request a consultation" secondary CTA (hybrid path).
  - `CART_CHECKOUT.md`: guest + account, server-side reprice, IDR/USD display, shipping, order creation, Snap token issuance.
  - `PAYMENT.md`: Midtrans Snap flow + manual bank-transfer flow (`awaiting_verification` → admin `verified_by`); webhook signature & idempotency.
  - `ORDERS.md`: status lifecycle (Glossary), order lookup for guests by number+email.
  - `ACCOUNT.md`: register/login, wishlist, order history, addresses.
  - `APPOINTMENT.md`: slot picker, conflict check (`409`), confirmation email, admin fulfilment.
  - `BOUTIQUE.md`, `CONSULTATION.md`, `EDITORIAL.md`, `AUTH.md` (RBAC matrix), `ADMIN.md` (modules + audit), `LEGAL_AND_FAQ.md` (privacy/terms/returns as `<TBD>` legal content).

### Step 14 — `FLOWS/`, `PATTERNS/`, `TASKS/`, `PROMPTS/`

- `FLOWS/`: `README.md` index; `BROWSING_FLOW.md` (land → collection/search → piece → cart/consultation); `CHECKOUT_FLOW.md` (cart → details → payment method → Snap/transfer → confirmation → fulfilment); `APPOINTMENT_FLOW.md` (boutique → slot → confirm → email → visit); `ADMIN_FLOW.md` (login → dashboard → manage pieces/orders/appointments/content). Each: overview mermaid + step tables + edge cases.
- `PATTERNS/`: `README.md` index (copy structure from master `PATTERNS/README.md`, listing the 6 patterns). `_TEMPLATE.md` verbatim from master. The 6 pattern files each carry: When to Use / When NOT / Structure / Implementation (real code snippet in this stack) / Anti-Pattern / Rules:
  - `SERVER_ACTION_PATTERN.md` — Zod-validated action, session/RBAC check, Prisma write, `revalidatePath`.
  - `PRISMA_MODEL_PATTERN.md` — model + relations + enums, migration/seed flow.
  - `COMPONENT_PATTERN.md` — server component + client island split, token-only styling.
  - `API_RESPONSE_PATTERN.md` — success/error envelope helpers in `src/lib/api/`.
  - `CATALOG_QUERY_PATTERN.md` — typed filter builder + pagination.
  - `PAYMENT_WEBHOOK_PATTERN.md` — signature verify → idempotency → state transition → email.
- `TASKS/`: `TODO.md` (High/Medium/Low + Completed, master format), `ROADMAP.md` (Phases + milestone table matching `PROJECT_CONTEXT` §Timeline + version history `v0.1.0`), `BUGS.md` (master bug template, empty active list).
- `PROMPTS/`: `UI_PROMPT.md`, `BACKEND_PROMPT.md`, `REFACTOR_PROMPT.md` — copy master prompt templates; swap placeholders to this stack (server actions, Prisma, Midtrans, appointment).

### Step 15 — Consistency pass

Re-read the full folder and enforce: (a) every file listed in `AGENTS.md` index exists and vice-versa; (b) every relative link target exists; (c) the design tokens in `UI_UX.md` exactly match the ones cited in `CODING_RULES.md`; (d) FR IDs in each `FEATURES/*.md` map to `PROJECT_CONTEXT.md`; (e) all `<TBD>` values are also listed in `CURRENT_STATE.md §Open Inputs`; (f) no fabricated business facts anywhere.

---

## Critical files & anchors

- `C:\Belajar Coding\Projects\.agents\` — master template (**read-only source**): `AGENTS.md`, `MANUAL.md`, `UI_UX.md`, `DECISIONS/_TEMPLATE.md`, `PATTERNS/README.md`, `FEATURES/AUTH.md`, `TASKS/BUGS.md`, `PROMPTS/*`.
- `C:\Belajar Coding\Projects\bifrost-website\.AGENTS\` — depth reference for the Next.js-stack docs (`CODING_RULES.md`, `STACK.md`, `ARCHITECTURE.md`, `DATABASE.md`, `API_CONTRACT.md`, `UI_UX.md`, `GLOSSARY.md`).
- `C:\Belajar Coding\Projects\PT Swastika Berdikari Website\.agents\` — depth reference for Tier-0 `CURRENT_STATE.md`, `FEATURES/README.md` index, and `DECISIONS/NNN-*.md` ADR style.
- `C:\Belajar Coding\Projects\bifrost-jewellery-website\.AGENTS\` — **the new output**.
- `C:\Belajar Coding\Projects\jwp-jewelry-management\` — domain-adjacent (jewellery SKU/stock concepts); optional terminology cross-check only.

## Verification

Docs-only deliverable — proof is structural + link integrity (no runtime to exercise):

1. **File inventory** — run from `bifrost-jewellery-website/`:
   `find .AGENTS -type f -name "*.md" | sort`
   Expect exactly the 55 files in the Directory layout — 11 root + 10 `DECISIONS/` + 15 `FEATURES/` + 5 `FLOWS/` + 8 `PATTERNS/` + 3 `TASKS/` + 3 `PROMPTS/`; verify count `find .AGENTS -type f -name "*.md" | wc -l` → `55`.
2. **No unfinished markers** — `grep -rn "<!-- TODO" .AGENTS` → must return nothing.
3. **All open inputs tracked** — every `<TBD` occurrence must also appear in `CURRENT_STATE.md`; check with:
   `diff <(grep -rho "<TBD[^>]*>" .AGENTS | sort -u) <(grep -o "TBD[^)]*" .AGENTS/CURRENT_STATE.md | sort -u)` (empty or explained).
4. **Relative links resolve** — node one-liner from `bifrost-jewellery-website/` over `.AGENTS/**/*.md`, for each `](...md...)` target assert `fs.existsSync(resolve(dirname(file), target))`; expect zero missing.
5. **Index ⇄ filesystem match** — every `*.md` named in `AGENTS.md` tables exists on disk, and every `.md` on disk is referenced by `AGENTS.md` (allowing `_TEMPLATE.md` and directory `README.md`).
6. **Mermaid sanity** — every ```` ```mermaid ```` fence has a matching closing fence and a non-empty first line (`graph|erDiagram|sequenceDiagram|flowchart`); count blocks per file with `grep -rc '```mermaid' .AGENTS`.
7. **Token consistency** — the color/typography tokens in `UI_UX.md §2–3` appear identically in `CODING_RULES.md`; `grep -n "ql-" .AGENTS/UI_UX.md .AGENTS/CODING_RULES.md` shows the same set.
8. **Palette contrast already verified** (this session, via the contrast tool): ink/canvas 16.42, muted/canvas 5.63, gold-deep/canvas 5.42, white/ink 16.42 (all AA); gold/canvas 3.69 → recorded as large-text-only rule. Re-check only if values change.

## Assumptions & contingencies

- **Assumption:** documents and public copy are English (ADR-008). *If the user instead wants Indonesian docs:* translate section bodies and headings, keep technical identifiers, file names, enum values, and ADR titles unchanged.
- **Assumption:** this task creates documentation only — no `package.json`, no `prisma/schema.prisma`, no `src/`. The database design lives as spec tables inside `DATABASE.md`. *If the user later wants a runnable scaffold:* that is a separate task; `DATABASE.md §4` and `STACK.md` env list are the inputs for it.
- **Assumption:** business facts not provided (domain, boutique address/phone/hours, contacts, catalog data, legal text) are `<TBD>` markers, never fabricated. *If the user supplies them during execution,* replace the markers and remove them from `CURRENT_STATE.md §Open Inputs`.
- **Assumption:** the master template at `Projects/.agents/` is the structural source. *If it has changed since this session,* follow its current structure but still do not modify it.
- **Contingency:** if `Projects/.agents/` or either sibling folder is unreadable at execution time, the Directory layout and Step specs in this plan are self-sufficient — build the files from this plan alone.
