# BlihOps V1 Content Feature — API Design

## 1. Purpose

REST API design for the managed website content feature, derived from
`01-Content-Model.md` and `02-Data-Model.md`. Defines routes, payloads,
validation, and business-rule semantics.

## 2. Conventions

- Feature folder `src/features/content/` mounted at
  `app.use('/api/v1/content', contentRouter)` in `src/app.ts`.
- Response envelopes (existing shared utils): `{ data }` via `sendSuccess`
  (single resources), `{ items, meta }` via `sendMany` (multiple resources;
  unpaginated lists return `meta: {}`), `{ error: { code, message, details? } }`
  via the error handler. Successful `DELETE` returns **204 No Content with no
  body**.
- **Public routes** (unauthenticated GETs) serve everything the web renders
  and live on the bare resource paths (`/api/v1/content/tags`).
  **Every admin action — including admin GETs — lives under the single
  `/api/v1/content/admin` subtree** (`/api/v1/content/admin/tags`,
  `/api/v1/content/admin/tags/:id`, …). One `admin.router.ts` applies
  `requireAuth` + `requireRole('admin')` once to the whole subtree, so no
  admin route can ever be registered without the guard.
- Public GETs return **only published or active content**. Because the
  `/admin` subtree is the only place filters like `status` exist, drafts can
  never leak through public paths.
- Public `/:slug` detail routes live under their resource path
  (`/api/v1/content/case-studies/:slug`) and can never collide with the
  `/admin` subtree.
- Bilingual resources (Case Studies, Insights, FAQs) return **both locales**
  in every payload; the web selects the variant matching its active locale.
  There is no `locale` query parameter.
- Zod schemas with `extendZodWithOpenApi`; routes registered in the OpenAPI
  registry.
- No free-text search in V1. Structural filters only.

## 3. File Structure

One feature folder; each resource is a self-contained mini-feature mirroring
the auth feature pattern, exporting a public router and an admin router:

```
src/features/content/
├── index.ts                 # exports contentRouter
├── content.router.ts        # public surface: mounts resource routers
│                            #   use('/tags', tagRouter), use('/case-studies', ...)
├── admin/
│   ├── index.ts             # exports adminContentRouter
│   └── admin.router.ts      # use(requireAuth, requireRole('admin')) ONCE,
│                            # then mounts admin routers:
│                            #   use('/tags', tagAdminRouter), use('/case-studies', ...)
├── common/
│   ├── html.ts              # sanitizeHtml helper
│   ├── schemas.ts           # pagination query schema, slug regex
│   └── helpers.ts           # JSON path slug uniqueness check etc.
└── <resource>/              # tags, categories, logos, testimonials,
    ├── index.ts             # services-hero, case-studies, insights,
    ├── router.ts            # public routes (GET /)                  careers, faqs
    ├── admin.router.ts      # admin routes (GET /, POST /, PATCH /:id, DELETE /:id)
    ├── controller.ts
    ├── service.ts
    ├── repository.ts
    └── schema.ts
```

`content.router.ts` mounts `admin.router.ts` at `/admin`; the guard lives
inside `admin.router.ts` itself, so it holds regardless of how the subtree is
mounted.

## 4. Route Inventory

### 4.1 Tags

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/tags` | public | all, sorted by name |
| GET | `/api/v1/content/admin/tags` | admin | all, sorted by name |
| POST | `/api/v1/content/admin/tags` | admin | `{ name }` |
| PATCH | `/api/v1/content/admin/tags/:id` | admin | rename |
| DELETE | `/api/v1/content/admin/tags/:id` | admin | 204 No Content; join rows cascade |

### 4.2 Categories

Identical shape to Tags (`/api/v1/content/categories`).

### 4.3 Trusted Logos

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/logos` | public | all, creation order |
| GET | `/api/v1/content/admin/logos` | admin | all, creation order |
| POST | `/api/v1/content/admin/logos` | admin | `{ imageUrl, alt }` |
| PATCH | `/api/v1/content/admin/logos/:id` | admin | |
| DELETE | `/api/v1/content/admin/logos/:id` | admin | |

### 4.4 Testimonials

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/testimonials` | public | all, creation order, `isPrimary` included |
| GET | `/api/v1/content/admin/testimonials` | admin | all, creation order |
| POST | `/api/v1/content/admin/testimonials` | admin | `{ avatarUrl, name, role, company, quote }` |
| PATCH | `/api/v1/content/admin/testimonials/:id` | admin | `isPrimary: true` clears the previous primary in a transaction |
| DELETE | `/api/v1/content/admin/testimonials/:id` | admin | blocked (409) if it is the current primary |

### 4.5 Services Hero Media (singleton)

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/services-hero` | public | the singleton record |
| GET | `/api/v1/content/admin/services-hero` | admin | same record (uniform admin view) |
| PUT | `/api/v1/content/admin/services-hero` | admin | full replace; upsert on fixed id `"global"` |

### 4.6 Case Studies

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/case-studies` | public | PUBLISHED only, newest first, paginated |
| GET | `/api/v1/content/case-studies/:slug` | public | PUBLISHED only; slug may match either locale; 404 otherwise |
| GET | `/api/v1/content/admin/case-studies` | admin | all statuses; filters `status`, `categoryId`; paginated; list items include `status` |
| GET | `/api/v1/content/admin/case-studies/:id` | admin | full detail: both locales, category, tags |
| POST | `/api/v1/content/admin/case-studies` | admin | creates a DRAFT; partial content allowed |
| PATCH | `/api/v1/content/admin/case-studies/:id` | admin | shared fields, or `{ locale, content }` for one locale, or `tags`/`categoryId` |
| POST | `/api/v1/content/admin/case-studies/:id/publish` | admin | validates both locales + shared fields, then flips status |
| POST | `/api/v1/content/admin/case-studies/:id/unpublish` | admin | flips to DRAFT |
| DELETE | `/api/v1/content/admin/case-studies/:id` | admin | |

### 4.7 Insights

Identical shape to Case Studies (`/api/v1/content/insights`). Admin list
items include `status`, like case studies.

### 4.8 Career Roles

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/careers` | public | `isActive` only, newest first, paginated |
| GET | `/api/v1/content/careers/:slug` | public | active only; 404 otherwise |
| GET | `/api/v1/content/admin/careers` | admin | all, filter `isActive`, paginated; list items include `isActive` |
| GET | `/api/v1/content/admin/careers/:id` | admin | |
| POST | `/api/v1/content/admin/careers` | admin | |
| PATCH | `/api/v1/content/admin/careers/:id` | admin | incl. `isActive` toggle |
| DELETE | `/api/v1/content/admin/careers/:id` | admin | |

### 4.9 Pilot FAQs

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/content/faqs` | public | active only, by `displayOrder` |
| GET | `/api/v1/content/admin/faqs` | admin | all, by `displayOrder` |
| GET | `/api/v1/content/admin/faqs/:id` | admin | |
| POST | `/api/v1/content/admin/faqs` | admin | |
| PATCH | `/api/v1/content/admin/faqs/:id` | admin | incl. `isActive`, `displayOrder` |
| DELETE | `/api/v1/content/admin/faqs/:id` | admin | |

Activating an FAQ requires both locales' question and answer non-empty.

## 5. Payload Shapes

### 5.1 Simple resources

List endpoints return the `{ items, meta }` envelope with an **empty
`meta` object** — no pagination in V1 for simple resources. Single-resource
endpoints use `{ data }`.

```json
// GET /content/logos
{ "items": [{ "id": "…", "imageUrl": "…", "alt": "…", "createdAt": "…" }], "meta": {} }

// GET /content/testimonials
{ "items": [{ "id": "…", "avatarUrl": "…", "name": "…", "role": "…",
              "company": "…", "quote": "…", "isPrimary": false }], "meta": {} }

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
  an extra fetch. Public list items do **not** include `status` — public
  lists only ever return published records.

Admin list (`GET /content/admin/case-studies`) — same item shape as the
public list plus `status` (the admin needs to distinguish drafts from
published records):

```json
{ "items": [
  { "id": "…",
    "slugs": { "en": "…", "de": "…" },
    "titles": { "en": "…", "de": "…" },
    "summaries": { "en": "…", "de": "…" },
    "client": "…",
    "category": { "id": "…", "name": "…" },
    "media": { "type": "image", "url": "…", "alt": "…" },
    "status": "DRAFT",
    "tags": [{ "id": "…", "name": "…" }],
    "createdAt": "…" }
], "meta": { "page": 1, "pageSize": 12, "total": 34, "totalPages": 3 } }
```

Admin detail (`GET /content/admin/case-studies/:id`):

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
{ "items": [{ "id": "…", "isActive": true, "displayOrder": 1,
              "content": {
                "en": { "question": "…", "answer": "…" },
                "de": { "question": "…", "answer": "…" }
              } }], "meta": {} }
```

### 5.4 Pagination

`page` (default 1), `pageSize` (default 12, max 100). Response meta:
`{ page, pageSize, total, totalPages }`. Applied to archive lists and all
admin lists. Simple resources (tags, categories, logos, testimonials, faqs)
are unpaginated — their lists use the `{ items, meta }` envelope with
`meta: {}`.

## 6. Validation Rules

### 6.1 Field limits

| Field | Rule |
|---|---|
| tag/category `name` | required, trimmed, ≤ 100 chars, unique |
| `imageUrl` / `avatarUrl` / `videoUrl` / `coverUrl` | required, ≤ 2048 chars |
| logo `alt` | required, ≤ 160 chars |
| media `alt` | optional, ≤ 160 chars |
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

`POST /admin/case-studies/:id/publish`:

1. Validate both locales: `title`, `slug`, `summary`, and body present and
   non-empty (case study: all three fixed sections; insight: ≥ 1 section).
2. Validate shared fields: `client` (case study) / `author` + `readTimeMinutes`
   (insight), `categoryId` set, `media.url` non-empty.
3. Check slug uniqueness per locale (both locales).
4. Set `status = PUBLISHED`.

Failures return `CONTENT_INCOMPLETE` with `details` listing the missing
fields. `POST /admin/case-studies/:id/unpublish` sets `status = DRAFT`
without validation.

### 7.2 Primary testimonial

- `PATCH /admin/testimonials/:id` with `isPrimary: true` runs a transaction:
  clear the existing primary, set the new one. At most one primary invariant.
- `DELETE /admin/testimonials/:id` on the current primary returns 409
  `CONTENT_PRIMARY_DELETE_BLOCKED` — Admin must select a replacement first.

### 7.3 Singleton hero media

- `PUT /api/v1/content/admin/services-hero` upserts the fixed-id record
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

| Code | Meaning | HTTP | Used |
|---|---|---|---|
| `CONTENT_INVALID_LOCALE` | locale value not `en`/`de` (admin PATCH) | 400 | yes |
| `CONTENT_SLUG_TAKEN` | slug already used in the target locale (publish / career slug) | 409 | yes |
| `CONTENT_INCOMPLETE` | publish/activate validation failed; `details` lists missing fields | 422 | yes |
| `CONTENT_PRIMARY_DELETE_BLOCKED` | deleting the current primary testimonial | 409 | yes |
| `CONTENT_NOT_FOUND` | content resource not found | 404 | reserved — generic `NOT_FOUND` is used |
| `CONTENT_VALIDATION_FAILED` | generic body validation failure | 400 | reserved — generic `VALIDATION_ERROR` (422) is used |

Existing `AppError` + error envelope handle transport; no new middleware.

## 9. OpenAPI

- Each resource registers its schemas and paths in the OpenAPI registry
  (`src/shared/openapi/registry.ts`) via `@asteasolutions/zod-to-openapi`,
  mirroring `openapi/auth.paths.ts`.
- Public paths and the `/admin` subtree are documented separately with their
  respective auth requirements.
