# BlihOps V1 Leads Feature — Lead Model

## 1. Purpose

This document is the source of truth for the leads feature: how inbound
client inquiries from the public site become leads, what a lead is, and the
rules that govern them.

Data modeling, API design, and request-flow architecture are documented
separately:

- `02-Data-Model.md` — Prisma schema, JSON payload shapes, constraints
- `03-API-Design.md` — endpoints, validation, webhook semantics
- `04-Architecture.md` — request flows, Cal.com integration, admin delivery

## 2. Scope

Leads are generated from the public marketing site (`blihops-web`) through
exactly three touchpoints:

| Source | Type | Capture mechanism |
|---|---|---|
| Contact form (`/contact`) | `CONTACT` | Form POST to the API |
| Pilot form (`/pilot`) | `PILOT` | Form POST to the API |
| Booked discovery call (Cal.com modal, site-wide) | `CALL` | Cal.com webhook to the API |

The admin app (`blihops-admin`) gets a Leads page where the team sees every
lead, filters it, changes its status, views its full details, and deletes it.

Out of scope (V1): notifications/email on new leads, public-facing lead
display, deduplication/merging of the same company across leads, editing
submitted lead data (a lead is a record of a submission, not a CRM contact).

## 3. Lead Type

Every lead carries exactly one type, set by the server based on the capture
mechanism — a client can never choose its own type:

| Type | Value | Notes |
|---|---|---|
| Contact | `CONTACT` | general enquiry via the contact form |
| Pilot | `PILOT` | free-pilot request via the pilot form |
| Call | `CALL` | booked discovery call, from any Cal.com booking site-wide |

The Cal.com modal is reachable from everywhere on the site (header, footer,
heroes, conversion CTAs, and after the pilot form submits) — **every booking
on the shared Cal.com link becomes a `CALL` lead**, regardless of which page
triggered it.

## 4. Lead Status

A lead progresses through a simple funnel. **Four statuses, three of them
user-touchable; `CONVERTED` and `CLOSED` are terminal:**

```text
NEW ──► CONTACTED ──► CONVERTED
                  └──► CLOSED
```

| Status | Meaning |
|---|---|
| `NEW` | default; received, not yet worked |
| `CONTACTED` | the team reached out to the lead |
| `CONVERTED` | the lead became a client / pilot / signed engagement (won) |
| `CLOSED` | no longer pursued — lost, unresponsive, cancelled call, spam-adjacent |

Rules:

- Every new lead starts as `NEW`.
- Status changes are admin-driven via PATCH, except the automatic
  `CALL`-specific transitions below.
- No transition matrix is enforced — admin may move a lead to any status
  (e.g. reopen a `CLOSED` lead that re-engages). `CONVERTED`/`CLOSED` are
  *semantically* terminal, not enforced.
- **Automatic transition (webhook only):** a cancelled Cal.com booking
  (`BOOKING_CANCELLED`) sets its matching `CALL` lead to `CLOSED`. A
  rescheduled booking (`BOOKING_RESCHEDULED`) updates the stored booking
  time/URL but does **not** change status.

## 5. Cross-cutting Decisions

### 5.1 Capture contracts

- **Contact / Pilot forms** — the web client POSTs the form payload to the
  API. The payload field names and limits mirror the existing client-side
  zod schemas (`blihops-web/src/lib/forms/contact.ts`, `pilot.ts`) exactly —
  the API is the authoritative validator and the web schema stays in sync.
- **Cal.com webhook** — the discovery-call event type
  (`yonatane-mk-sa4cic/discovery-call`) is configured with a webhook
  pointing at the API. The API verifies the `X-Cal-Signature-256` HMAC
  signature before trusting anything. The booking form popup questions
  (company, main challenge, how did you hear about us, team size) arrive
  in the webhook payload's `responses` object and are mapped onto the
  lead (§7.3, §8).

### 5.2 Locale

- The web sends the visitor's active locale (`en` | `de`) with each form
  submission; it is stored in `details.locale` so the team knows which
  language the lead used. Defaults to `en` when absent.
- Cal.com bookings carry no locale — omitted.

### 5.3 Spam defense (public form endpoints)

- **Honeypot:** both public forms include a hidden `website` field
  (visually hidden, `tabIndex=-1`, `aria-hidden`, `autocomplete="off"`).
  The API treats any non-empty `website` value as bot traffic: it responds
  with the normal success envelope but **does not persist** the lead.
- **Rate limiting:** the public form endpoints are rate-limited per IP
  via `express-rate-limit` (see API design §7.2). The webhook endpoint is not IP-rate-limited — its
  gate is signature verification — but is protected by the same zod
  validation.

### 5.4 Admin capabilities (Leads page)

- List all leads with filters: `type`, `status`, free-text search
  (name / email / company).
- Detail view: every field plus the type-specific `details` payload.
- Change status (PATCH).
- Delete a lead (with confirmation).

### 5.5 No notifications

V1 has no email/Slack notification on new leads — the team checks the Leads
page. Revisit after the pilot.

## 6. Model

| Field | Type | Notes |
|---|---|---|
| `id` | string (cuid) | |
| `type` | `CONTACT` \| `PILOT` \| `CALL` | set by server, never by client |
| `status` | `NEW` \| `CONTACTED` \| `CONVERTED` \| `CLOSED` | default `NEW` |
| `fullName` | string | attendee name for `CALL` (first attendee) |
| `workEmail` | string | attendee email for `CALL` (first attendee) |
| `company` | string? | `CALL`: from the booking form's Company question (null if unanswered); contact: optional; pilot: required |
| `details` | JSON object | type-specific payload, see §7 |
| `createdAt` | datetime | |
| `updatedAt` | datetime | reflects status changes and webhook updates |

## 7. Type-specific `details` payloads

### 7.1 `CONTACT`

```json
{
  "topic": "Outsourcing services",
  "message": "…",
  "locale": "en"
}
```

| Key | Required | Notes |
|---|---|---|
| `topic` | yes | one of the four contact topics (API design §6.2) |
| `message` | yes | the enquiry text |
| `locale` | yes | `en` \| `de`, default `en` |

### 7.2 `PILOT`

```json
{
  "service": "Customer support",
  "challenge": "…",
  "volume": "100–500 tasks per month",
  "timeline": "Within 30 days",
  "context": "…",
  "locale": "en"
}
```

| Key | Required | Notes |
|---|---|---|
| `service` | yes | one of the six service options |
| `challenge` | yes | the operational challenge |
| `volume` | yes | one of the five volume options |
| `timeline` | yes | one of the four timeline options |
| `context` | no | optional extra context |
| `locale` | yes | `en` \| `de`, default `en` |

### 7.3 `CALL`

```json
{
  "bookingTime": "2026-08-20T14:00:00.000Z",
  "timezone": "Europe/Berlin",
  "bookingUrl": "https://cal.com/…/…",
  "calBookingUid": "abc123…",
  "challenge": "Ticket backlog is growing faster than the team can handle.",
  "hearAbout": "LinkedIn",
  "teamSize": "10–50",
  "cancelledAt": null
}
```

| Key | Required | Notes |
|---|---|---|
| `bookingTime` | yes | ISO datetime; Cal.com `startTime`; updated on reschedule |
| `timezone` | yes | Cal.com attendee timezone |
| `bookingUrl` | yes | the Cal.com booking URL; updated on reschedule |
| `calBookingUid` | yes | Cal.com booking UID — the join key for cancel/reschedule events |
| `challenge` | no | answer to "What's the main challenge you'd like to solve?" (popup question) |
| `hearAbout` | no | answer to "How did you hear about us?" (`LinkedIn` \| `Google` \| `Referral` \| `Event` \| `Other`) |
| `teamSize` | no | answer to "Team size?" (`<10` \| `10–50` \| `50–200` \| `200+`) |
| `cancelledAt` | no | ISO datetime, set when `BOOKING_CANCELLED` arrives |

`company` is null for `CALL` leads. `fullName`/`workEmail` come from the
booking's **first attendee**.

## 8. Business Rules

1. **Server-set type** — clients can only hit the endpoint for the type
   they represent; no generic "create lead with type X" route exists.
2. **Status starts `NEW`** on every create path (form or webhook).
3. **Cancel closes** — `BOOKING_CANCELLED` for a known `calBookingUid`
   sets that lead's status to `CLOSED` and stamps `details.cancelledAt`.
4. **Reschedule updates, never closes** — `BOOKING_RESCHEDULED` updates
   `bookingTime` and `bookingUrl` on the matched lead; status untouched.
5. **Unknown booking UID on cancel/reschedule** — the event is logged and
   ignored (200 to Cal.com); no lead is created from a cancellation.
6. **Idempotent webhook handling** — the same booking UID never creates
   two `CALL` leads (Cal.com retries deliveries).
7. **Honeypot never persists** — non-empty `website` → success response,
   no row.
8. **Leads are immutable submissions** — V1 admin can only change `status`
   (and delete); `details`, `fullName`, `workEmail`, `company` are not
   editable. Corrections = delete + note, or future CRM edit feature.
9. **No duplicate suppression** — the same company may appear as multiple
   leads (contact + pilot + call). Accepted for V1.
10. **Popup answers map onto the lead** — Cal.com `responses` for the
    booking form questions are mapped at webhook time: Company →
    `company` column, main challenge → `details.challenge`, how did you
    hear → `details.hearAbout`, team size → `details.teamSize`. Missing or
    unanswered questions are simply omitted — never fail the event.

## 9. Carried to Later Steps

- Exact zod schemas, limits, and option enums (API design step).
- Webhook signature verification mechanics (API design step).
- Pagination/filter shapes for the admin list (API design step).
- TanStack Query keys and optimistic update flow (architecture step).
