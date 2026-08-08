# BlihOps V1 Content Feature — Architecture

## 1. Purpose

Request-flow and system architecture for the managed website content feature,
derived from `01-Content-Model.md`, `02-Data-Model.md`, and
`03-API-Design.md`. Describes how the frontends, API, database, and blob
storage interact for every flow in the feature.

## 2. Component Diagram

```
+--------------------------------+        +-----------------------------------+
|  blihops-web (Vercel)          |        |  blihops-api (Render)             |
|  - Public pages (SSR + ISR)    |        |  - /api/v1/content router         |
|  - server components fetch     |  HTTPS |  - requireAuth / requireRole      |
|    public GETs (no auth)       | -----> |  - content service + repository   |
|  - TanStack Query client cache |        +-----------------------------------+
                                          |             |                    |
                                          | Prisma      | never touches     |
                                          v             v blob SDK          v
+--------------------------------+  +----------------+   +-----------------------+
|  blihops-admin (Vercel)        |  |  Neon (Postgres)|   |  Vercel Blob          |
|  - TanStack Query + apiFetch   |  |  - content     |   |  - images / videos    |
|  - /admin calls (session       |--|    tables      |   |  - uploaded by admin  |
|    cookie via apiFetch)        |  |                |   |    route handler      |
|  - /api/uploads route handler  |  +----------------+   +-----------------------+
|    (blob put(), token server-  |
|    side only)                  |
+--------------------------------+
```

No new infrastructure. Both frontends are on Vercel, the API on Render, data
in Neon; `CORS_ORIGIN` already lists both frontend origins (Deployment §5).

## 3. Frontend Data Layer

### 3.1 Transport: `apiFetch`

The existing `apiFetch` wrapper (identical in both apps) remains the single
HTTP transport: JSON envelopes, `credentials: 'include'` for the session
cookie, timeout, and retry/backoff. TanStack Query composes on top of it; no
new fetch library.

### 3.2 Admin: TanStack Query

- `@tanstack/react-query` v5; `QueryClientProvider` at the admin root layout.
- Server state is always fetched through TanStack Query.
- Query keys per resource:

  ```ts
  ['content', 'tags']
  ['content', 'categories']
  ['content', 'logos']
  ['content', 'testimonials']
  ['content', 'services-hero']
  ['content', 'case-studies', 'admin', { page, pageSize, status, categoryId }]
  ['content', 'case-studies', 'detail', id]
  ['content', 'insights', 'admin', { page, pageSize, status, categoryId }]
  ['content', 'insights', 'detail', id]
  ['content', 'careers', 'admin', { page, isActive }]
  ['content', 'careers', 'detail', id]
  ['content', 'faqs', 'admin']
  ['content', 'faqs', 'detail', id]
  ```

- Mutations wrap `apiFetch` with `useMutation` and invalidate on success:
  - create / update / delete / publish / unpublish → invalidate the list and
    detail keys for that resource
  - testimonial primary change → invalidate `['content','testimonials']`
  - singleton hero update → invalidate `['content','services-hero']`
- Defaults: `staleTime` ~30 s; retries inherited from `apiFetch` (2 retries,
  backoff) — no extra query-level retry needed.

### 3.3 Web: ISR server components + TanStack Query client cache

- Public content pages (Home trust sections, Case Studies, Insights, Careers,
  Pilot FAQs, hero media) are **server components fetching through the API
  with ISR**: `export const revalidate = 300`. Bounded staleness — an admin
  publish appears on the public site within the revalidation window (~5 min);
  no webhook, no build-time coupling.
- **TanStack Query v5 at the web root** provides the client-side cache for
  public content. Server components prefetch the data and hydrate the query
  cache (`hydrate`/`dehydrate`), so initial HTML is ISR-rendered while
  subsequent interactions and navigations read from the client cache without
  new server round-trips.
- Client-side fetches use `NEXT_PUBLIC_API_URL`; server-side ISR fetches use
  the server-only `API_URL` (Deployment §6).
- Public query keys:

  ```ts
  ['content', 'logos']
  ['content', 'testimonials']
  ['content', 'services-hero']
  ['content', 'case-studies', 'public', { page }]
  ['content', 'case-studies', 'detail', slug]
  ['content', 'insights', 'public', { page }]
  ['content', 'insights', 'detail', slug]
  ['content', 'careers', 'public', { page }]
  ['content', 'careers', 'detail', slug]
  ['content', 'faqs']
  ```

- Pages pick the locale variant of bilingual payloads with a small
  `contentByLocale(content, locale)` helper — the API always returns both
  locales.
- Default `staleTime` (~5 min) aligns with the ISR window so the server and
  client cache layers agree.

## 4. Request Flows

### 4.1 Admin CRUD (create / update / delete)

```text
Admin UI (client component)
  → useMutation(apiFetch('PATCH /api/v1/content/case-studies/admin/:id', body))
  → content.router
  → router.use('/admin', requireAuth, requireRole('admin'))   // subtree guard
  → controller (zod parse, 400 CONTENT_VALIDATION_FAILED)
  → service
      - sanitize-html on every HTML string (both locales)
      - slug-per-locale uniqueness check (Prisma JSON path filter, excl. self)
  → repository (Prisma)
  → Neon
  → { data } envelope
  → mutation success → invalidate ['content','case-studies', ...] keys
```

### 4.2 Admin publish (Case Studies / Insights)

```text
POST /api/v1/content/case-studies/admin/:id/publish
  → subtree guard (admin)
  → service:
      1. validate both locales: title, slug, summary, body non-empty
         (case study: 3 fixed sections; insight: ≥ 1 section)
      2. validate shared: client / author + readTimeMinutes, categoryId,
         media.url
      3. slug uniqueness per locale (both locales)
      → failure: 422 CONTENT_INCOMPLETE with details
  → repository: status = PUBLISHED
  → activity record
  → { data }
  → invalidate list + detail keys
```

`POST /admin/:id/unpublish` flips `status = DRAFT` without validation.

### 4.3 Primary testimonial

```text
PATCH /api/v1/content/testimonials/admin/:id  { isPrimary: true }
  → transaction: clear existing primary, set new one  (at-most-one invariant)
DELETE /api/v1/content/testimonials/admin/:id  (current primary)
  → 409 CONTENT_PRIMARY_DELETE_BLOCKED until a replacement is chosen
```

### 4.4 Public delivery

```text
Web server component (revalidate = 300)
  → apiFetch('GET /api/v1/content/case-studies?page=1&pageSize=12')
    (server-side, API_URL, no auth)
  → repository: WHERE status = PUBLISHED, ORDER BY createdAt DESC, pagination
  → { items, meta } (both locales embedded)
  → contentByLocale(item, locale) picks titles/slugs/summaries for render
  → Vercel cache serves subsequent requests within the ISR window
  → data hydrated into the TanStack Query cache for client-side reuse
```

Simple resources (logos, testimonials, hero media, FAQs) are fetched the same
way without pagination; public careers/insights follow the same pattern as
case studies.

### 4.5 Upload flow

```text
Admin UI (upload control)
  → POST /api/uploads (admin Next.js route handler, server-side)
  → @vercel/blob put(buffer, { access: 'public',
      allowedContentTypes: [image/jpeg, image/png, image/webp, image/svg+xml,
                            video/mp4, video/webm],
      maximumSizeInBytes: image 5 MB / video 50 MB })
  → returns blob URL
  → admin UI saves the URL into the content payload
  → PATCH /api/v1/content/... persists the URL string
```

- `BLOB_READ_WRITE_TOKEN` exists only in the admin app env — never in the
  browser bundle, never in the API.
- The API validates URL length and media `type` enum; it never performs
  uploads.

### 4.6 Error flow

```text
AppError (service/repository)
  → errorHandler middleware
  → { error: { code, message, details? } }  (existing envelope)
  → apiFetch throws ApiError(status, code, message)
  → TanStack Query surfaces it; mutations show retryable errors
```

## 5. Enforcement-Layer Map

| Concern | Layer |
|---|---|
| Payload shape, field limits, slug regex | zod schemas in each resource's `schema.ts` |
| HTML sanitization, publish completeness, primary invariant, slug uniqueness, reserved `admin` slug | service layer |
| JSON path filters, pagination, PUBLISHED/active filters | repository layer |
| Unique tag/category names, unique career slug, cascade join deletes, category `onDelete: Restrict` | database |
| Auth on all admin routes | subtree guard `router.use('/admin', ...)` |

## 6. Security Notes

- Draft-revealing filters (`status`, `categoryId`, all-records views) exist
  only under the authenticated `/admin` subtree; public paths structurally
  cannot expose drafts or inactive content.
- Sanitize-on-write: HTML is cleaned in the service before persist; the DB is
  the single trusted source on read (public paths never re-sanitize).
- The blob token never leaves the admin server.
- Reserved slug `admin` prevents shadowing the admin subtree.
- Cookie-based sessions; CORS with credentials for both frontend origins.

## 7. Env & Wiring

| App | Env | Status |
|---|---|---|
| admin | `BLOB_READ_WRITE_TOKEN` | **to add** (admin env only) |
| admin | `NEXT_PUBLIC_API_URL`, `API_URL` | already configured |
| web | `API_URL` (server-side ISR fetches) | already configured |
| api | `CORS_ORIGIN` (both origins) | already configured |

Dependencies to add:

- admin: `@tanstack/react-query`
- web: `@tanstack/react-query`

## 8. Out of Scope (V1)

- Webhook-based cache invalidation from API to web
- CDN or edge caching beyond Vercel's ISR
- Any blob SDK usage inside the API
