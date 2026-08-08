# BlihOps V1 Content Feature — Data Model

## 1. Purpose

Prisma schema design for the managed website content feature, derived from
`01-Content-Model.md`. Documents the localization strategy, every model,
the JSON payload shapes, and how each product rule is enforced.

## 2. Conventions

- Models append to the existing `blihops-api/prisma/schema.prisma` (single
  schema file, `prisma-client` generator already configured — no config
  change required).
- Match existing model style: `PascalCase` models, `camelCase` fields,
  `String @id @default(cuid())`, `createdAt @default(now())`,
  `updatedAt @updatedAt`, no `@@map`.
- Every content model carries `createdAt`/`updatedAt` (omitted below for
  brevity).

## 3. Localization Strategy

**Decision: JSON content objects (Option A).** Bilingual models store their
localized content in one `Json` column shaped `{ en: {...}, de: {...} }`
instead of per-locale child rows.

Rationale:

- Publish/unpublish applies to both locales together — one row means one
  atomic status flip, no cross-row consistency.
- Admin edits one record with EN/DE tabs; drafts may hold a partial object.
- Fewer tables, no joins, natural fit for the admin-tabs mental model.
- Shape, completeness, and slug-per-locale uniqueness are validated in the
  application layer (zod), which is the established pattern. DB-native slug
  uniqueness (the only real advantage of per-locale rows) is not worth the
  join and upsert cost for an admin-only CMS in V1.

This applies to CaseStudy, Insight, and PilotFaq. `PilotFaq` uses the same
pattern for one consistent bilingual convention across the feature.

## 4. Schema

```prisma
enum ContentStatus {
  DRAFT
  PUBLISHED
}

model Tag {
  id          String        @id @default(cuid())
  name        String        @unique
  createdAt   DateTime      @default(now())
  updatedAt   DateTime      @updatedAt
  caseStudies CaseStudyTag[]
  insights    InsightTag[]
}

model CaseStudyTag {
  caseStudyId String    @id
  tagId       String    @id
  caseStudy   CaseStudy @relation(fields: [caseStudyId], references: [id], onDelete: Cascade)
  tag         Tag       @relation(fields: [tagId], references: [id], onDelete: Cascade)
}

model InsightTag {
  insightId String   @id
  tagId     String   @id
  insight   Insight  @relation(fields: [insightId], references: [id], onDelete: Cascade)
  tag       Tag      @relation(fields: [tagId], references: [id], onDelete: Cascade)
}

model Category {
  id          String      @id @default(cuid())
  name        String      @unique
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt
  caseStudies CaseStudy[]
  insights    Insight[]
}

model TrustedLogo {
  id        String   @id @default(cuid())
  imageUrl  String
  alt       String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Testimonial {
  id        String   @id @default(cuid())
  avatarUrl String
  name      String
  role      String
  company   String
  quote     String
  isPrimary Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model ServicesHeroMedia {
  id        String   @id @default("global")
  videoUrl  String
  coverUrl  String
  altLabel  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model CaseStudy {
  id         String        @id @default(cuid())
  client     String
  categoryId String?
  category   Category?     @relation(fields: [categoryId], references: [id], onDelete: Restrict)
  media      Json
  status     ContentStatus @default(DRAFT)
  content    Json
  createdAt  DateTime      @default(now())
  updatedAt  DateTime      @updatedAt
  tags       CaseStudyTag[]
}

model Insight {
  id              String        @id @default(cuid())
  author          String
  categoryId      String?
  category        Category?     @relation(fields: [categoryId], references: [id], onDelete: Restrict)
  readTimeMinutes Int
  media           Json
  status          ContentStatus @default(DRAFT)
  content         Json
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
  tags            InsightTag[]
}

model CareerRole {
  id               String   @id @default(cuid())
  title            String
  slug             String   @unique
  department       String
  location         String
  employmentType   String
  summary          String
  overview         String[]
  responsibilities String[]
  requirements     String[]
  isActive         Boolean  @default(false)
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
}

model PilotFaq {
  id           String   @id @default(cuid())
  content      Json
  isActive     Boolean  @default(false)
  displayOrder Int      @default(0)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}
```

### 4.1 JSON payload shapes

`CaseStudy.content`:

```json
{
  "en": {
    "title": "…",
    "slug": "…",
    "summary": "…",
    "body": {
      "challenge": "<tiptap-html>",
      "approach": "<tiptap-html>",
      "outcome": "<tiptap-html>"
    }
  },
  "de": { "title": "…", "slug": "…", "summary": "…", "body": { … } }
}
```

`Insight.content`:

```json
{
  "en": {
    "title": "…",
    "slug": "…",
    "excerpt": "…",
    "body": [{ "section": "…", "content": "<tiptap-html>" }]
  },
  "de": { "title": "…", "slug": "…", "excerpt": "…", "body": [] }
}
```

`PilotFaq.content`:

```json
{
  "en": { "question": "…", "answer": "…" },
  "de": { "question": "…", "answer": "…" }
}
```

`media` (CaseStudy and Insight):

```json
{ "type": "image", "url": "…", "alt": "…" }
```

`alt` is optional; `type` is `"image" | "video"`.

## 5. Design Notes

- **Categories and Tags** are plain admin-managed vocabularies (`name`
  unique). No enums — categories are no longer predefined.
- **Category is nullable** so a draft can be created without one; publish
  validation requires it. `onDelete: Restrict` prevents deleting a category
  still assigned to a record.
- **Tags** use explicit join tables (`CaseStudyTag`, `InsightTag`) with
  composite primary keys (`caseStudyId` + `tagId`), which prevent duplicate
  membership. Both relations use `onDelete: Cascade` — deleting a Tag (or a
  Case Study/Insight) auto-unlinks the assignment.
- **Singleton hero media** via fixed id `"global"` — save is an upsert, no
  index tricks.
- **Primary testimonial (at-most-one)**: enforced in the application layer
  with a transactional update (set new → clear previous). Optional hardening:
  a Postgres partial unique index via raw SQL migration:

  ```sql
  CREATE UNIQUE INDEX "Testimonial_primary_idx" ON "Testimonial" ((true)) WHERE "isPrimary";
  ```

- **Case study body** keys (`challenge`/`approach`/`outcome`) and their
  rendered labels are fixed constants; the DB only stores the rich-text HTML
  per key.
- **Career slug** is globally unique — careers are English-only, so there is
  no locale dimension.
- `ServicesHeroMedia.updatedAt` is exposed to the API as `lastUpdatedAt`.

## 6. Product Rule → Enforcement Map

| Rule | Enforcement |
|---|---|
| Publish requires both locales | App layer: zod completeness check on publish action |
| Publish/unpublish both locales together | Single `status` column on the record row (atomic flip) |
| Slugs unique per locale | App layer: Prisma JSON path filter (`content: { path: ["de","slug"], equals }`) |
| Drafts may be incomplete | `content` Json accepts partial objects; completeness validated only at publish |
| At most one primary testimonial | App-layer transaction (+ optional partial unique index) |
| Primary testimonial undeletable | App-layer delete guard |
| Hero media singleton | Fixed id `"global"`, upsert |
| Category required to publish | App-layer publish validation (FK nullable for drafts) |
| Only active careers public | `isActive` filter on public queries |
| FAQ active requires both locales | App-layer check on activation |
| Tag deletion unlinks records | Explicit join tables with `onDelete: Cascade` |
| Category deletion blocked while assigned | `onDelete: Restrict` |
| HTML sanitization | Content service sanitizes Tiptap output before persist |
| Media URLs only | Admin uploads to Vercel Blob; API stores URL strings |

## 7. Migration Plan

Single migration applying all content models:

```bash
pnpm db:migrate -- --name add_content_models
```

(`prisma migrate dev`; generates the migration and applies it locally.)

If the partial unique index is adopted, include the raw SQL in the same
migration. Production applies the committed migration via
`prisma migrate deploy` (see `03-Engineering/Deployment.md`).
