# BlihOps V1 Content Feature — Content Model

## 1. Purpose

This document is the source of truth for the managed website content feature
(the "CMS"). It defines the content models and the rules that govern them,
based on the content-model inventory (planning step 1).

Data modeling, API design, and request-flow architecture are documented
separately:

- `02-Data-Model.md` — Prisma schema, localization strategy, constraints
- `03-API-Design.md` — endpoints, validation, publish semantics
- `04-Architecture.md` — request flows, upload flow, public delivery

## 2. Scope

Admin manages the following website content:

| Area | Content |
|---|---|
| Home | Trusted Logos, Testimonials |
| Services | Hero media (singleton) |
| Case Studies | Bilingual records (EN + DE) |
| Insights | Bilingual records (EN + DE) |
| Careers | English-only roles |
| Pilot | Bilingual FAQs (EN + DE) |
| Shared | Tags and Categories (shared vocabularies for Case Studies and Insights) |

Out of scope (stays static code, not CMS-managed): the five service offering
pages, About/How-We-Work copy, navigation, locale strings, and any
general-purpose page building. Existing `blihops-web/src/content` and
`LogoCloud.tsx` data are placeholder UI content and dictate nothing for this
feature.

## 3. Cross-cutting Decisions

### 3.1 Rich text

- Body content is edited with Tiptap in the Admin app and stored as HTML.
- The API sanitizes HTML server-side (`sanitize-html`) before persisting.
- The public web renders only sanitized content.
- No Markdown, no structured-block format.

### 3.2 Media

- Storage: Vercel Blob (free tier). The Admin app performs uploads directly
  via `@vercel/blob`; the API only stores the returned URL strings and never
  touches the blob SDK.
- Media object shape: `{ type: 'image' | 'video', url, alt? }`.
  `alt` is optional but recommended for accessibility.
- Per-type media:
  - Trusted Logo — image
  - Testimonial — avatar image (required)
  - Services hero — video + cover image
  - Case Study / Insight hero — image **or** video

### 3.3 Localization

- Case Studies and Insights: one record with English and German content.
  Publish and unpublish apply to both locales together. Slugs are unique
  within their locale.
- Pilot FAQs: EN and DE Q/A on one entry; both locales are required before the
  entry can be active.
- Careers: English only.
- Logos, Testimonials, hero media, Tags, and Categories: not localized.

### 3.4 States

| Model | State model |
|---|---|
| Case Study / Insight | `DRAFT` \| `PUBLISHED`; drafts may be incomplete |
| Career Role | active toggle; only active roles are public |
| Pilot FAQ | active toggle; requires both locales |
| Trusted Logo / Testimonial | none — all entries are shown |
| Services hero media | singleton, always live |

### 3.5 Ordering

| Model | Ordering |
|---|---|
| Case Study / Insight / Career Role | newest first (by creation) |
| Pilot FAQ | explicit display order |
| Trusted Logo / Testimonial | creation order |
| Tag | n/a |

## 4. Models

### 4.1 Tag

Reusable vocabulary shared by Case Studies and Insights.

| Field | Type | Notes |
|---|---|---|
| name | string | unique, trimmed |

Rules:

- Many-to-many with Case Studies and Insights; optional on both.
- No slug, no tag archive pages in V1 — public display is chips only.
- Admin can create, edit, and delete tags.

### 4.2 Category

One shared list used by both Case Studies and Insights.

| Field | Type | Notes |
|---|---|---|
| name | string | unique, trimmed |

Rules:

- Admin-managed: created, edited, and deleted from Admin; no predefined list.
- A Case Study or Insight is assigned exactly one Category.
- Not localized.

### 4.3 TrustedLogo

| Field | Type | Notes |
|---|---|---|
| imageUrl | string | required |
| alt | string | required, accessibility text |

Rules:

- No active state and no display order: all logos are shown in creation order.

### 4.4 Testimonial

| Field | Type | Notes |
|---|---|---|
| avatarUrl | string | required |
| name | string | required |
| role | string | required |
| company | string | required |
| quote | string | required |
| isPrimary | boolean | exactly one active primary at any time |

Rules:

- The first testimonial is not automatically primary; Admin marks one
  explicitly.
- Marking a testimonial as primary clears the previous primary (one active
  primary invariant).
- The current primary **cannot be deleted** until Admin selects a replacement.
- No active toggle and no display order: testimonials render in creation order.

### 4.5 ServicesHeroMedia (singleton)

| Field | Type | Notes |
|---|---|---|
| videoUrl | string | |
| coverUrl | string | |
| altLabel | string | accessible media label / alt text |
| lastUpdatedAt | datetime | updated on save |

Rules:

- Singleton: exactly one record exists; Admin replaces the video or cover and
  saves.

### 4.6 CaseStudy

Localized fields (per locale):

| Field | Type | Notes |
|---|---|---|
| title | string | required to publish |
| slug | string | unique within the locale |
| summary | string | required to publish |
| body | object | fixed 3-section structure (see below) |

Body structure — fixed keys and fixed rendered labels; Admin fills content only:

```json
{
  "challenge": "<tiptap-html>",
  "approach": "<tiptap-html>",
  "outcome": "<tiptap-html>"
}
```

Shared fields:

| Field | Type | Notes |
|---|---|---|
| client | string | |
| category | Category | required to publish; one shared admin-managed list (§4.2) |
| media | `{ type, url, alt? }` | image or video |
| tags | Tag[] | optional |
| status | `DRAFT` \| `PUBLISHED` | |

Rules:

- Both locales must validate before publication; drafts may be incomplete.
- Publication and unpublication apply to both locales together.
- Slug uniqueness is per locale.
- No featured flag.

### 4.7 Insight

Localized fields (per locale):

| Field | Type | Notes |
|---|---|---|
| title | string | required to publish |
| slug | string | unique within the locale |
| excerpt | string | required to publish |
| body | `{ section, content }[]` | unlimited, admin-defined sections (see below) |

Body structure — an ordered array of sections with admin-authored titles;
supports in-page section navigation:

```json
[
  { "section": "Start with a decision", "content": "<tiptap-html>" },
  { "section": "Define the operating boundary", "content": "<tiptap-html>" }
]
```

Shared fields:

| Field | Type | Notes |
|---|---|---|
| author | string | |
| category | Category | required to publish; one shared admin-managed list (§4.2) |
| readTimeMinutes | number | stored as minutes; rendering formats the label |
| media | `{ type, url, alt? }` | image or video |
| tags | Tag[] | optional |
| status | `DRAFT` \| `PUBLISHED` | |

Rules:

- Both locales must validate before publication; drafts may be incomplete.
- Publication and unpublication apply to both locales together.
- Slug uniqueness is per locale.
- No featured flag.

### 4.8 CareerRole

| Field | Type | Notes |
|---|---|---|
| title | string | required |
| slug | string | for URL routing |
| department | string | |
| location | string | |
| employmentType | string | |
| summary | string | |
| overview | string[] | paragraph list |
| responsibilities | string[] | |
| requirements | string[] | |
| isActive | boolean | only active roles are public |

Rules:

- English only.
- No featured flag.

### 4.9 PilotFaq

| Field | Type | Notes |
|---|---|---|
| enQuestion | string | |
| enAnswer | string | |
| deQuestion | string | |
| deAnswer | string | |
| isActive | boolean | requires both locales |
| displayOrder | number | |

Rules:

- Both locales are required before an FAQ can become active.
- Only active FAQs appear on the Pilot page for the selected locale.

## 5. Content Safety Rules

- Destructive deletion requires confirmation.
- Upload failures preserve unsaved form data where possible.
- Published or active content changes create activity records (audit, see
  PRD §13).
- Missing bilingual content cannot be published with an English fallback.

## 6. Categories

Categories are not predefined. Admin manages a single shared list through the
Category model (§4.2); both Case Studies and Insights assign exactly one
Category from that list.

## 7. Carried to Later Steps

- Slug generation and validation rules (data model / API steps).
- Tag deletion semantics when assignments exist.
- Per-locale required-field lists for publish validation (API step).
- Upload size and type limits per media field (API step).
