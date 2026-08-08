# BlihOps V1 Content Feature — API Design

## 1. Purpose

REST API design for the managed website content feature, derived from
`01-Content-Model.md` and `02-Data-Model.md`. Defines routes, payloads,
validation, and business-rule semantics.

## 2. Conventions

- Feature folder `src/features/content/` mounted at
  `app.use('/api/v1/content', contentRouter)` in `src/app.ts`.
- Response envelopes (existing shared utils): `{ data }` via `sendSuccess`,
  `{ items, meta }` via `sendMany`, `{ error: { code, message, details? } }`
  via the error handler.
- **Public routes** (unauthenticated GETs) serve everything the web renders.
  **Every admin action — including admin GETs — lives under an `/admin`
  sub-path per resource**, so the auth guard can be applied once per subtree
  (`router.use('/admin', requireAuth, requireRole('admin'))`) instead of
  per-route.
- Public GETs return **only published or active content**. Because the
  `/admin` subtree is the only place filters like `status` exist, drafts can
  never leak through public paths.
- Admin routes are registered **before** public `/:slug` routes so the
  literal path `admin` can never be captured as a slug; the slug `admin` is
  additionally reserved in validation (see §6.2).
- Bilingual resources (Case Studies, Insights, FAQs) return **both locales**
  in every payload; the web selects the variant matching its active locale.
  There is no `locale` query parameter.
- Zod schemas with `extendZodWithOpenApi`; routes registered in the OpenAPI
  registry.
- No free-text search in V1. Structural filters only.

## 3. File Structure

One feature folder; each resource is a self-contained mini-feature mirroring
the auth feature pattern:

```
src/features/content/
├── index.ts                 # exports contentRouter
├── content.router.ts        # mounts the 9 sub-routers
├── common/
│   ├── html.ts              # sanitizeHtml helper
│   ├── schemas.ts           # pagination query schema, slug regex
│   └── helpers.ts           # JSON path slug uniqueness check etc.
└── <resource>/              # tags, categories, logos, testimonials,
    ├── index.ts             # services-hero, case-studies, insights,
    ├── router.ts            # careers, faqs
    ├── controller.ts
    ├── service.ts
    ├── repository.ts
    └── schema.ts
```

## 4. Route Inventory

### 4.1 Tags

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/tags` | public | all, sorted by name |
| GET | `/api/v1/content/tags/admin` | admin | all, sorted by name |
| POST | `/api/v1/content/tags/admin` | admin | `{ name }` |
| PATCH | `/api/v1/content/tags/admin/:id` | admin | rename |
| DELETE | `/api/v1/content/tags/admin/:id` | admin | join rows cascade |

### 4.2 Categories

Identical shape to Tags (`/api/v1/content/categories`).

### 4.3 Trusted Logos

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/logos` | public | all, creation order |
| GET | `/api/v1/content/logos/admin` | admin | all, creation order |
| POST | `/api/v1/content/logos/admin` | admin | `{ imageUrl, alt }` |
| PATCH | `/api/v1/content/logos/admin/:id` | admin | |
| DELETE | `/api/v1/content/logos/admin/:id` | admin | |

### 4.4 Testimonials

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/testimonials` | public | all, creation order, `isPrimary` included |
| GET | `/api/v1/content/testimonials/admin` | admin | all, creation order |
| POST | `/api/v1/content/testimonials/admin` | admin | `{ avatarUrl, name, role, company, quote }` |
| PATCH | `/api/v1/content/testimonials/admin/:id` | admin | `isPrimary: true` clears the previous primary in a transaction |
| DELETE | `/api/v1/content/testimonials/admin/:id` | admin | blocked (409) if it is the current primary |

### 4.5 Services Hero Media (singleton)

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/services-hero` | public | the singleton record |
| GET | `/api/v1/content/services-hero/admin` | admin | same record (uniform admin view) |
| PUT | `/api/v1/content/services-hero/admin` | admin | full replace; upsert on fixed id `"global"` |

### 4.6 Case Studies

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/case-studies` | public | PUBLISHED only, newest first, paginated |
| GET | `/api/v1/content/case-studies/:slug` | public | PUBLISHED only; slug may match either locale; 404 otherwise |
| GET | `/api/v1/content/case-studies/admin` | admin | all statuses; filters `status`, `categoryId`; paginated |
| GET | `/api/v1/content/case-studies/admin/:id` | admin | full detail: both locales, category, tags |
| POST | `/api/v1/content/case-studies/admin` | admin | creates a DRAFT; partial content allowed |
| PATCH | `/api/v1/content/case-studies/admin/:id` | admin | shared fields, or `{ locale, content }` for one locale, or `tags`/`categoryId` |
| POST | `/api/v1/content/case-studies/admin/:id/publish` | admin | validates both locales + shared fields, then flips status |
| POST | `/api/v1/content/case-studies/admin/:id/unpublish` | admin | flips to DRAFT |
| DELETE | `/api/v1/content/case-studies/admin/:id` | admin | |

### 4.7 Insights

Identical shape to Case Studies (`/api/v1/content/insights`).

### 4.8 Career Roles

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/careers` | public | `isActive` only, newest first, paginated |
| GET | `/api/v1/content/careers/:slug` | public | active only; 404 otherwise |
| GET | `/api/v1/content/careers/admin` | admin | all, filter `isActive`, paginated |
| GET | `/api/v1/content/careers/admin/:id` | admin | |
| POST | `/api/v1/content/careers/admin` | admin | |
| PATCH | `/api/v1/content/careers/admin/:id` | admin | incl. `isActive` toggle |
| DELETE | `/api/v1/content/careers/admin/:id` | admin | |

### 4.9 Pilot FAQs

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/faqs` | public | active only, by `displayOrder` |
| GET | `/api/v1/content/faqs/admin` | admin | all, by `displayOrder` |
| GET | `/api/v1/content/faqs/admin/:id` | admin | |
| POST | `/api/v1/content/faqs/admin` | admin | |
| PATCH | `/api/v1/content/faqs/admin/:id` | admin | incl. `isActive`, `displayOrder` |
| DELETE | `/api/v1/content/faqs/admin/:id` | admin | |

Activating an FAQ requires both locales' question and answer non-empty.

## 5. Payload Shapes

### 5.1 Simple resources

```json
// GET /content/logos
{ "data": [{ "id": "…", "imageUrl": "…", "alt": "…", "createdAt": "…" }] }

// GET /content/testimonials
{ "data": [{ "id": "…", "avatarUrl": "…", "name": "…", "role": "…",
             "company": "…", "quote": "…", "isPrimary": false }] }

// GET /content/services-hero
{ "data": { "id": "global", "videoUrl": "…", "coverUrl": "…",
            "altLabel": "…", "updatedAt": "…" } }
```

### 5.2 Case Study / Insight

Public list item (`GET /content/case-studies`):

```json
{ "items": [
  { "id": "…", "slug": "…", "title": "…", "summary": "…", "client": "…",
    "category": { "id": "…", "name": "…" },
    "media": { "type": "image", "url": "…", "alt": "…" },
    "tags": [{ "id": "…", "name": "…" }],
    "createdAt": "…" }
], "meta": { "page": 1, "pageSize": 12, "total": 34, "totalPages": 3 } }
```

`slug`/`title`/`summary` in list items reflect the **current request's
locale-aware content only if single-locale fields are requested** — in V1
public list items expose the EN variant values alongside both full bodies
in detail; the web selects by locale. Exact DTO mapping:

- Public **detail** returns `content` with **both locales**; the web picks
  `content.en` or `content.de`.
- Public **list** items include `slug`, `title`, `summary`, `category`,
  `media`, `tags`, `createdAt`. The web derives list display from the locale
  it is rendering; list DTOs carry both locales' `slug`/`title`/`summary`
  where ambiguity matters (`slugs: { en, de }`, `titles: { en, de }`,
  `summaries: { en, de }`) so a German archive renders German text without
  an extra fetch.

Admin detail (`GET /content/case-studies/admin/:id`):

```json
{ "data": {
  "id": "…", "client": "…",
  "category": { "id": "…", "name": "…" },
  "media": { "type": "video", "url": "…", "alt": null },
  "status": "DRAFT",
  "tags": [{ "id": "…", "name": "…" }],
  "content": {
    "en": { "title": "…", "slug": "…", "summary": "…",
            "body": { "challenge": "<html>", "approach": "<html>", "outcome": "<html>" } },
    "de": { "title": "…", "slug": "…", "summary": "…", "body": { … } }
  },
  "createdAt": "…", "updatedAt": "…"
} }
```

### 5.3 FAQ

```json
{ "data": [{ "id": "…", "isActive": true, "displayOrder": 1,
             "content": {
               "en": { "question": "…", "answer": "…" },
               "de": { "question": "…", "answer": "…" }
             } }] }
```

### 5.4 Pagination

`page` (default 1), `pageSize` (default 12, max 100). Response meta:
`{ page, pageSize, total, totalPages }`. Applied to archive lists and all
admin lists. Simple resources (tags, categories, logos, testimonials, faqs)
are unpaginated.

## 6. Validation Rules

### 6.1 Field limits

| Field | Rule |
|---|---|
| tag/category `name` | required, trimmed, ≤ 100 chars, unique |
| `imageUrl` / `avatarUrl` / `videoUrl` / `coverUrl` | required, ≤ 2048 chars |
| logo `alt`, media `alt` | optional, ≤ 160 chars |
| testimonial `name`/`role`/`company` | required, ≤ 100 chars |
| testimonial `quote` | required, ≤ 2000 chars |
| case study `client` | required to publish, ≤ 200 chars |
| insight `author` | required to publish, ≤ 100 chars |
| insight `readTimeMinutes` | required to publish, ≥ 1 |
| career fields | required on create, limits per field (title ≤ 150, others ≤ 500, list entries ≤ 500) |
| faq `question`/`answer` | required to activate, ≤ 500 / ≤ 4000 |
| HTML body content | sanitized, ≤ 200 KB per section |

### 6.2 Slugs

- Regex: `^[a-z0-9]+(?:-[a-z0-9]+)*$` (lowercase kebab).
- The literal slug `admin` is **reserved** (rejected in validation) so the
  public `/:slug` route can never shadow the admin subtree, independent of
  route ordering.
- Career slug: globally unique (DB constraint).
- Case study / insight slug: unique **per locale** — app-layer check via
  Prisma JSON path filter before create/update/publish:
  `content: { path: ["de", "slug"], equals: value }`, excluding self.
- Public detail lookup matches the slug against either locale's slug path.

### 6.3 Upload limits

Enforced at Admin upload time (Vercel Blob client-side):

- Images: ≤ 5 MB, `image/jpeg | image/png | image/webp | image/svg+xml`
- Videos: ≤ 50 MB, `video/mp4 | video/webm`

The API additionally validates URL length and media `type` enum. The API
never performs uploads.

## 7. Business-Rule Semantics

### 7.1 Publish (Case Studies / Insights)

`POST /admin/:id/publish`:

1. Validate both locales: `title`, `slug`, `summary`, and body present and
   non-empty (case study: all three fixed sections; insight: ≥ 1 section).
2. Validate shared fields: `client` (case study) / `author` + `readTimeMinutes`
   (insight), `categoryId` set, `media.url` non-empty.
3. Check slug uniqueness per locale (both locales).
4. Set `status = PUBLISHED`.

Failures return `CONTENT_INCOMPLETE` with `details` listing the missing
fields. `POST /admin/:id/unpublish` sets `status = DRAFT` without validation.

### 7.2 Primary testimonial

- `PATCH /admin/:id` with `isPrimary: true` runs a transaction: clear the
  existing primary, set the new one. At most one primary invariant.
- `DELETE /admin/:id` on the current primary returns 409
  `CONTENT_PRIMARY_DELETE_BLOCKED` — Admin must select a replacement first.

### 7.3 Singleton hero media

- `PUT /api/v1/content/services-hero/admin` upserts the fixed-id record
  (`"global"`); full replace of `videoUrl`, `coverUrl`, `altLabel`.

### 7.4 Sanitization

- Every Tiptap HTML string (case study body sections, insight body sections,
  FAQ answers) passes through `sanitize-html` in the service layer **before
  persist** — both locales, on create, update, and publish.
- Public read paths never re-sanitize; the DB is the single trusted source.

### 7.5 Activity records

- Content changes that reach published/active state, plus creation,
  publication, activation, ordering, and deletion, create activity records
  (PRD §13). The `Activity` model is outside this feature's data model and is
  recorded through the existing activity mechanism (implementation detail to
  be wired when the activity feature lands).

## 8. Error Codes

New codes added to `src/shared/errors/errorCodes.ts`:

| Code | Meaning | HTTP |
|---|---|---|
| `CONTENT_INVALID_LOCALE` | locale value not `en`/`de` (admin PATCH) | 400 |
| `CONTENT_SLUG_TAKEN` | slug already used in the target locale | 409 |
| `CONTENT_INCOMPLETE` | publish/activate validation failed; `details` lists missing fields | 422 |
| `CONTENT_PRIMARY_DELETE_BLOCKED` | deleting the current primary testimonial | 409 |
| `CONTENT_NOT_FOUND` | admin resource or public slug not found | 404 |
| `CONTENT_VALIDATION_FAILED` | generic body validation failure | 400 |

Existing `AppError` + error envelope handle transport; no new middleware.

## 9. OpenAPI

- Each resource registers its schemas and paths in the OpenAPI registry
  (`src/shared/openapi/registry.ts`) via `@asteasolutions/zod-to-openapi`,
  mirroring `openapi/auth.paths.ts`.
- Public and admin-prefixed paths are documented separately with their
  respective auth requirements.
