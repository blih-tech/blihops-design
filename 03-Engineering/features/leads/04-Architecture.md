# BlihOps V1 Leads Feature — Architecture

## 1. Purpose

Request-flow and system architecture for the leads feature, derived from
`01-Lead-Model.md`, `02-Data-Model.md`, and `03-API-Design.md`. Describes
how the web forms, Cal.com, the API, and the admin app interact for every
flow in the feature.

## 2. Component Diagram

```
+--------------------------------+         +---------------------------+
|  blihops-web (Vercel)          |         |  Cal.com                  |
|  - /contact form (POST)        |  HTTPS  |  - discovery call booking |
|  - /pilot form (POST)          | ------> |  - webhook (X-Cal-       |
|  - Cal.com modal (embed)       |         |    Signature-256)         |
+--------------------------------+         +------------+--------------+
                                                        | webhook (HTTPS)
+--------------------------------+         +------------v--------------+
|  blihops-admin (Vercel)        |         |  blihops-api (Render)     |
|  - Leads page (TanStack Query) |  HTTPS  |  - /api/v1/leads router   |
|  - filters, detail, status,    | ------> |  - requireAuth/requireRole|
|    delete via apiFetch         |         |  - leads service/repo     |
+--------------------------------+         +------------+--------------+
                                                        | Prisma
                                                        v
                                                +----------------+
                                                | Neon (Postgres) |
                                                |  - Lead table   |
                                                +----------------+
```

New external integration: **Cal.com webhooks** (outbound from Cal.com to
the API). No other new infrastructure — both frontends stay on Vercel, the
API on Render, data in Neon.

## 3. Public Form Flows (web)

### 3.1 Contact form

```text
User submits /contact form
  → client-side zod validation (existing ContactForm.tsx schema)
  → apiFetch POST /api/v1/leads/contact
      body: fullName, workEmail, company?, topic, message, locale,
            website (honeypot, hidden input)
  → 201 { data }  → success screen (existing submitted state)
  → 4xx/5xx       → inline error state on the form (keep user input)
```

Changes to `ContactForm.tsx`:

- Add the hidden honeypot input (`website`, `tabIndex={-1}`,
  `aria-hidden`, `autoComplete="off"`, visually hidden via CSS).
- Replace the current `console.info` submit handler with the apiFetch call.
- Add an error state (currently only `submitted` exists) — on failure show
  a retryable error near the submit button and keep the form values.
- Send the active locale (`useLocale()` from next-intl) as `locale`.

### 3.2 Pilot form

Same as §3.1 against `POST /api/v1/leads/pilot` (payload per API design
§5.2), plus the existing behavior is preserved: on success the Cal.com
modal opens (`getCalApi` + `cal('modal')`) and the success screen shows.
If the API call fails, the modal does **not** open — the error state
shows instead.

## 4. Cal.com Webhook Flow

### 4.1 Booking created

```text
Visitor books a discovery call (any BookCallButton / pilot modal)
  → Cal.com creates the booking
  → Cal.com fires BOOKING_CREATED webhook → POST /api/v1/leads/webhooks/calcom
  → webhook.ts verifies X-Cal-Signature-256 (HMAC-SHA256, raw body,
    CALCOM_WEBHOOK_SECRET)
  → service.handleCalWebhook:
      - dispatch on triggerEvent
      - BOOKING_CREATED: findByCalBookingUid(payload.uid) — exists? → 200 no-op
        else create Lead { type: CALL, status: NEW,
          fullName: attendees[0].name, workEmail: attendees[0].email,
          company: responses['Company'] ?? null,
          details: { bookingTime: startTime, timezone, bookingUrl: url,
                     calBookingUid: uid,
                     challenge/hearAbout/teamSize from responses,
                     cancelledAt: null } }
  → 200 OK
```

### 4.2 Booking cancelled / rescheduled

```text
Visitor cancels or reschedules the call
  → Cal.com fires BOOKING_CANCELLED / BOOKING_RESCHEDULED
  → same endpoint + signature verification
  → findByCalBookingUid(payload.uid):
      CANCELLED  → update: status = CLOSED,
                   details.cancelledAt = now        (status untouched on
      RESCHEDULED → update: details.bookingTime,     RESCHEDULED)
                   details.bookingUrl
  → not found → log + 200 OK (never create from cancel/reschedule)
```

Cal.com retries non-2xx deliveries; the handler always returns 200 for
recognized-but-unmappable events so Cal.com stops retrying.

## 5. Admin Flows (blihops-admin)

### 5.1 Transport & queries

- Existing `apiFetch` + TanStack Query v5 (`QueryClientProvider` already at
  the admin root). `credentials: 'include'` for the session cookie.
- Query keys:

  ```ts
  ['leads', { type, status, q, page }]        // list
  ['leads', 'detail', id]                     // detail
  ```

- Mutations:
  - status change → `PATCH /api/v1/leads/admin/:id` → invalidate the list
    key **and** the detail key
  - delete → `DELETE /api/v1/leads/admin/:id` → invalidate the list key
- Defaults: `staleTime` ~30 s; retries inherited from `apiFetch`.

### 5.2 Leads page

New route under `(dashboard)/leads/` (page + section component per the
house pattern; nav entry added to `lib/nav.ts`):

- **Toolbar**: type filter (segmented control or select: All / Contact /
  Pilot / Call), status filter (All / New / Contacted / Converted /
  Closed), search input (`q`, debounced ~300 ms).
- **Table**: company, fullName, workEmail, type badge, status badge,
  createdAt. Sorted newest first. Pagination controls (page/pageSize).
- **Detail**: dialog (house pattern) fetching `['leads','detail',id]` —
  renders the shared fields plus a type-specific details panel:
  - CONTACT: topic, message, locale
  - PILOT: service, challenge, volume, timeline, context, locale
  - CALL: booking time (formatted with timezone), booking URL (link),
    company, challenge, hearAbout, teamSize (when present), cancelledAt
    if set
- **Status change**: select in the table row and/or detail dialog →
  optimistic PATCH (rollback on error).
- **Delete**: confirm dialog → DELETE → 204 → invalidate list, close
  dialog.

## 6. Enforcement-Layer Map

| Concern | Layer |
|---|---|
| Payload shape, field limits, option enums, honeypot emptiness | zod schemas in `schema.ts` |
| Type assignment, status defaults, idempotency check, cancel/reschedule semantics, no-op on unknown UID | service layer |
| Filters, pagination, JSON path lookup on `calBookingUid`, `createdAt DESC` | repository layer |
| Webhook signature (HMAC-SHA256, raw body) | `webhook.ts` before service |
| Rate limiting on public form endpoints | `express-rate-limit` middleware on the two form routes |
| Auth on all admin routes | subtree guard `router.use('/admin', requireAuth, requireRole('admin'))` |
| Status column enum, indexes | database |

## 7. Security Notes

- Public surface is write-only: two form POSTs (rate-limited, honeypot)
  and one signed webhook. No lead data is ever readable without admin auth.
- Webhook trust is cryptographic — `X-Cal-Signature-256` verified with
  constant-time compare; `CALCOM_WEBHOOK_SECRET` lives in the API env only
  (Render dashboard), never in any frontend.
- Admin subtree guard is applied once in `admin.router.ts` — no admin route
  can be registered outside it.
- Honeypot responses are indistinguishable from success (no bot feedback
  loop).

## 8. Env & Wiring

| App | Env | Status |
|---|---|---|
| api | `CALCOM_WEBHOOK_SECRET` | **to add** (Render dashboard + local `.env`) |
| api | `CORS_ORIGIN` | already configured (web + admin origins) |
| web | `NEXT_PUBLIC_API_URL` | already configured |
| admin | `NEXT_PUBLIC_API_URL` | already configured |
| Cal.com | webhook URL → `https://<api>/api/v1/leads/webhooks/calcom` | **to configure** in the Cal.com dashboard (event type webhook) with the same secret |
| api | `express-rate-limit` (npm dependency) | **to add** |

## 9. Out of Scope (V1)

- Email/Slack notifications on new leads
- Public display of leads anywhere on the web site
- Deduplication/merging leads for the same company
- Editing submission data (details/fullName/email/company) from admin
- Lead assignment/ownership, internal notes, activity history
- Cal.com webhook events beyond create/cancel/reschedule
  (e.g. `MEETING_ENDED`, `FORM_SUBMITTED`)

## 10. Accepted V1 Debt

- **No dedupe**: the same company can appear as multiple leads (contact +
  pilot + call). The team handles this manually until a client entity
  exists.
- **`calBookingUid` in JSON**: lookups by booking UID are JSON path
  queries. Fine at V1 volume; promote to a real indexed column if webhook
  traffic grows or booking events become hot.
- **Webhook delivery ordering**: if a `BOOKING_CANCELLED` somehow arrives
  before its `BOOKING_CREATED`, the cancel is a logged no-op and the later
  create leaves a NEW lead. Cal.com's event ordering makes this unlikely;
  accepted.
