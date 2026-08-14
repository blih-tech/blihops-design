# BlihOps V1 Leads Feature — API Design

## 1. Purpose

REST API design for the leads feature, derived from `01-Lead-Model.md` and
`02-Data-Model.md`. Defines routes, payloads, validation, webhook
semantics, and business-rule behavior.

## 2. Conventions

- Feature folder `src/features/leads/` mounted at
  `app.use('/api/v1/leads', leadsRouter)` in `src/app.ts`.
- Response envelopes (existing shared utils): `{ data }` via `sendSuccess`
  (single resources), `{ items, meta }` via `sendMany` (lists), `{ error:
  { code, message, details? } }` via the error handler. Successful `DELETE`
  returns **204 No Content with no body**.
- **Admin routes live under the single `/api/v1/leads/admin` subtree**
  (`admin.router.ts` applies `requireAuth` + `requireRole('admin')` once).
- Zod schemas with `extendZodWithOpenApi`; routes registered in the OpenAPI
  registry.
- Public routes are **unauthenticated POSTs only** — there are no public
  GETs; leads are never exposed to the public site.
- Error codes: reuse the shared `ErrorCodes` (`NOT_FOUND`,
  `VALIDATION_ERROR`, `TOO_MANY_REQUESTS`, `UNAUTHORIZED`, `FORBIDDEN`).
  Add `LEAD_NOT_FOUND` for lead lookups and `LEAD_WEBHOOK_INVALID` for
  failed webhook verification (API design §8).

## 3. File Structure

```text
src/features/leads/
├── index.ts                 # exports leadsRouter
├── leads.router.ts          # public routes:
│                            #   POST /contact, POST /pilot, POST /webhooks/calcom
├── admin/
│   ├── index.ts             # exports adminLeadsRouter
│   └── admin.router.ts      # use(requireAuth, requireRole('admin')) ONCE,
│                            # then: GET /, GET /:id, PATCH /:id, DELETE /:id
├── schema.ts                # zod schemas: contact/pilot payloads, status,
│                            #   webhook payload, pagination, option enums
├── controller.ts            # public handlers (parse → service)
├── service.ts               # createLead, listLeads, getLead, updateStatus,
│                            #   deleteLead, handleCalWebhook
├── repository.ts            # prisma: create, findMany w/ filters +
│                            #   pagination, findUnique, update, delete,
│                            #   findByCalBookingUid
├── webhook.ts               # Cal.com signature verification + event mapping
└── paths.ts                 # OpenAPI path registration (public + admin)
```

## 4. Route Inventory

### 4.1 Public — form submissions

| Method | Path | Auth | Notes |
|---|---|---|---|
| POST | `/api/v1/leads/contact` | public | creates a `CONTACT` lead |
| POST | `/api/v1/leads/pilot` | public | creates a `PILOT` lead |
| POST | `/api/v1/leads/webhooks/calcom` | webhook signature | Cal.com event receiver |

### 4.2 Admin — Leads management

| Method | Path | Auth | Notes |
|---|---|---|---|
| GET | `/api/v1/leads/admin` | admin | list, filters + pagination |
| GET | `/api/v1/leads/admin/:id` | admin | full detail incl. `details` |
| PATCH | `/api/v1/leads/admin/:id` | admin | `{ status }` only |
| DELETE | `/api/v1/leads/admin/:id` | admin | 204 No Content |

## 5. Payload Shapes

### 5.1 `POST /api/v1/leads/contact`

Request:

```json
{
  "fullName": "Jane Doe",
  "workEmail": "jane@acme.com",
  "company": "Acme GmbH",
  "topic": "Outsourcing services",
  "message": "We need a support pod for our SaaS.",
  "locale": "en",
  "website": ""
}
```

- `company` optional; `locale` optional (default `en`); `website` is the
  **honeypot** — must be empty, otherwise the request is treated as bot
  traffic (see §7.1).
- `topic` must be one of the four contact topics (§6.2).

Response `201 Created`:

```json
{ "data": { "id": "cm0…", "type": "CONTACT", "status": "NEW" } }
```

### 5.2 `POST /api/v1/leads/pilot`

Request:

```json
{
  "fullName": "Jane Doe",
  "workEmail": "jane@acme.com",
  "company": "Acme GmbH",
  "service": "Customer support",
  "challenge": "Our ticket backlog is growing faster than the team can handle.",
  "volume": "100–500 tasks per month",
  "timeline": "Within 30 days",
  "context": "Team of 12, EU timezone overlap needed.",
  "locale": "en",
  "website": ""
}
```

- `company` **required** (mirrors the web form); `context` optional;
  `website` honeypot as above.

Response `201 Created`:

```json
{ "data": { "id": "cm0…", "type": "PILOT", "status": "NEW" } }
```

### 5.3 `POST /api/v1/leads/webhooks/calcom`

Cal.com sends `{ triggerEvent, payload }` bodies with the
`X-Cal-Signature-256` header. The API accepts only these events:

| Trigger | Action |
|---|---|
| `BOOKING_CREATED` | create `CALL` lead (idempotent by `calBookingUid`) |
| `BOOKING_CANCELLED` | close matched lead + stamp `details.cancelledAt` |
| `BOOKING_RESCHEDULED` | update matched lead's `bookingTime`/`bookingUrl` |

Response: always `200 OK` (empty body or `{ data: { ok: true } }`) once the
event is processed. Unknown events → `200 OK`, ignored. Failed signature →
`401` `LEAD_WEBHOOK_INVALID`.

**Matching for cancel/reschedule** — repository lookup by JSON path:
`details: { path: ['calBookingUid'], equals: payload.uid }`. Not found →
log + `200 OK` (never create from a cancel/reschedule).

**Popup question answers** — the booking form questions arrive in
`payload.responses` (keys = the question labels as configured in
Cal.com). Mapping: Company → `company` column; main challenge →
`details.challenge`; how did you hear about us → `details.hearAbout`;
team size → `details.teamSize`. Missing keys are omitted — never fail
the event for an unanswered question.

### 5.4 `GET /api/v1/leads/admin`

Query params: `type` (`CONTACT|PILOT|CALL`), `status`
(`NEW|CONTACTED|CONVERTED|CLOSED`), `q` (free-text), `page` (default 1),
`pageSize` (default 20, max 100). Sorted `createdAt DESC`.

Response:

```json
{
  "items": [
    {
      "id": "cm0…",
      "type": "PILOT",
      "status": "NEW",
      "fullName": "Jane Doe",
      "workEmail": "jane@acme.com",
      "company": "Acme GmbH",
      "createdAt": "2026-08-13T10:00:00.000Z",
      "updatedAt": "2026-08-13T10:00:00.000Z"
    }
  ],
  "meta": { "page": 1, "pageSize": 20, "total": 34, "totalPages": 2 }
}
```

List items omit `details` (kept for the detail endpoint) — the table only
needs the shared columns.

### 5.5 `GET /api/v1/leads/admin/:id`

Response:

```json
{
  "data": {
    "id": "cm0…",
    "type": "PILOT",
    "status": "NEW",
    "fullName": "Jane Doe",
    "workEmail": "jane@acme.com",
    "company": "Acme GmbH",
    "details": {
      "service": "Customer support",
      "challenge": "…",
      "volume": "100–500 tasks per month",
      "timeline": "Within 30 days",
      "context": "…",
      "locale": "en"
    },
    "createdAt": "…",
    "updatedAt": "…"
  }
}
```

### 5.6 `PATCH /api/v1/leads/admin/:id`

Request — **only `status` is accepted** (zod strips/400s anything else):

```json
{ "status": "CONTACTED" }
```

Response:

```json
{ "data": { "id": "cm0…", "type": "PILOT", "status": "CONTACTED" } }
```

### 5.7 `DELETE /api/v1/leads/admin/:id`

`204 No Content`. `404 LEAD_NOT_FOUND` if the id doesn't exist.

## 6. Validation Rules

### 6.1 Shared field limits

| Field | Rule |
|---|---|
| `fullName` | required, trimmed, 2–80 chars |
| `workEmail` | required, valid email, ≤ 254 chars |
| `company` | contact: optional, ≤ 120; pilot: required, 2–120 chars |
| `locale` | optional, `en` \| `de`, default `en` |
| `website` | honeypot — must be empty string; non-empty = bot |

### 6.2 Option enums (exact string values, mirroring the web forms)

Contact `topic`:

`Outsourcing services`, `AI and workflow automation`, `Partnership`,
`General enquiry`

Pilot `service`:

`Customer support`, `Back-office operations`, `Data processing`,
`IT and software support`, `AI and workflow automation`, `Not sure yet`

Pilot `volume`:

`Under 100 tasks per month`, `100–500 tasks per month`,
`500–2,000 tasks per month`, `More than 2,000 tasks per month`,
`It varies or is not measured yet`

Pilot `timeline`:

`As soon as possible`, `Within 30 days`, `Within 1–3 months`,
`Just exploring for now`

Limits:

| Field | Rule |
|---|---|
| `topic` | required, one of the enum above |
| `message` | required, trimmed, 20–1500 chars |
| `service` | required, one of the enum above |
| `challenge` | required, trimmed, 20–1200 chars |
| `volume` | required, one of the enum above |
| `timeline` | required, one of the enum above |
| `context` | optional, trimmed, ≤ 1000 chars |

> **Enum sync contract:** these exact strings are the API ↔ web contract
> (like the icon whitelist). Changing an option means updating the web form
> option list, the API zod enum, and the admin detail labels together.

## 7. Business-Rule Semantics

### 7.1 Honeypot (public form endpoints)

- The `website` field is part of the payload schema and validated as an
  **empty string**.
- If it is non-empty → the controller returns the same `201` success
  envelope **without calling the service** (no DB write). No error, no
  signal to the bot.
- If the field is missing entirely → treated as empty (legit client).

### 7.2 Rate limiting (public form endpoints)

- Implemented with **`express-rate-limit`** (new API dependency), applied
  per-route to `POST /leads/contact` and `POST /leads/pilot`:
  `windowMs: 10 * 60 * 1000` (10 min), `limit: 20`, keyed by IP.
- Over the limit → `429 TOO_MANY_REQUESTS` using the shared error
  envelope (`{ error: { code, message } }`) — the middleware's default
  handler is replaced with one that emits the standard envelope.
- The webhook endpoint is **not** IP-rate-limited (Cal.com servers) — its
  gate is the HMAC signature.

### 7.3 Cal.com webhook handling

1. **Verify** — HMAC-SHA256 of the raw request body using
   `CALCOM_WEBHOOK_SECRET`; compare against `X-Cal-Signature-256`
   (constant-time compare). Failure → `401 LEAD_WEBHOOK_INVALID`, no
   processing.
2. **Dispatch by `triggerEvent`** (see §5.3).
3. **`BOOKING_CREATED`** — extract first attendee name/email, map
   `startTime` → `bookingTime`, `timeZone` → `timezone`, `url` →
   `bookingUrl`, `uid` → `calBookingUid`, and map popup answers from
   `payload.responses`: Company → `company`, main challenge →
   `details.challenge`, how did you hear → `details.hearAbout`, team size
   → `details.teamSize` (missing keys omitted). Idempotency: check
   `findByCalBookingUid` first; if a lead already exists, `200 OK` without
   inserting (Cal.com retries deliveries).
4. **`BOOKING_CANCELLED`** — match by `calBookingUid`; set
   `status = CLOSED` and `details.cancelledAt = now`; `200 OK`. No match →
   log + `200 OK`.
5. **`BOOKING_RESCHEDULED`** — match by `calBookingUid`; update
   `details.bookingTime` and `details.bookingUrl`; status untouched;
   `200 OK`. No match → log + `200 OK`.

Webhook body parsing uses `express.json()` — the signature is computed over
the **raw body**, so the route must capture `req.rawBody` (existing
`express.json({ verify })` pattern in `src/app.ts`).

### 7.4 Admin status change

- PATCH accepts `{ status }` only. Any status value in the enum is
  accepted (no transition matrix — admin may reopen).
- Zod-invalid bodies → `400 VALIDATION_ERROR` (existing zod-boundary
  behavior).

## 8. Error Codes

| Code | Status | When |
|---|---|---|
| `LEAD_NOT_FOUND` | 404 | lead id unknown (detail/patch/delete) |
| `LEAD_WEBHOOK_INVALID` | 401 | Cal.com signature verification failed |
| `VALIDATION_ERROR` | 400/422 | malformed payload at the zod boundary (existing behavior) |
| `TOO_MANY_REQUESTS` | 429 | rate limit exceeded on public form endpoints |
| `UNAUTHORIZED` / `FORBIDDEN` | 401/403 | admin subtree guard |

## 9. Seed Data

A dev seed script (`src/shared/scripts/seed-leads.ts`) creates a few sample
leads (one of each type, mixed statuses) for the admin UI, mirroring the
`seed-services.ts` pattern. Not run in production.
