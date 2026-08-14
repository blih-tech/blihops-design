# BlihOps V1 Leads Feature — Data Model

## 1. Purpose

Prisma schema design for the leads feature, derived from
`01-Lead-Model.md`. Documents the model, the JSON payload shapes, and how
each product rule is enforced.

## 2. Conventions

- Model appends to the existing `blihops-api/prisma/schema.prisma` (single
  schema file, `prisma-client` generator already configured — no config
  change required).
- Match existing model style: `PascalCase` models, `camelCase` fields,
  `String @id @default(cuid())`, `createdAt @default(now())`,
  `updatedAt @updatedAt`, no `@@map`.
- Enum style matches existing enums (`ContentStatus` → `LeadType`,
  `LeadStatus`).

## 3. Schema

```prisma
enum LeadType {
  CONTACT
  PILOT
  CALL
}

enum LeadStatus {
  NEW
  CONTACTED
  CONVERTED
  CLOSED
}

model Lead {
  id        String     @id @default(cuid())
  type      LeadType
  status    LeadStatus @default(NEW)
  fullName  String
  workEmail String
  company   String?
  details   Json       @default("{}")
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt

  @@index([type, status])
  @@index([createdAt])
}
```

## 4. JSON payload shapes

### 4.1 `Lead.details` — CONTACT

```json
{
  "topic": "Outsourcing services",
  "message": "We need a support pod for our SaaS.",
  "locale": "en"
}
```

### 4.2 `Lead.details` — PILOT

```json
{
  "service": "Customer support",
  "challenge": "Ticket backlog is growing…",
  "volume": "100–500 tasks per month",
  "timeline": "Within 30 days",
  "context": "Team of 12, EU timezone overlap needed.",
  "locale": "en"
}
```

### 4.3 `Lead.details` — CALL

```json
{
  "bookingTime": "2026-08-20T14:00:00.000Z",
  "timezone": "Europe/Berlin",
  "bookingUrl": "https://cal.com/yonatane-mk-sa4cic/discovery-call/abc123",
  "calBookingUid": "abc123",
  "challenge": "Ticket backlog is growing faster than the team can handle.",
  "hearAbout": "LinkedIn",
  "teamSize": "10–50",
  "cancelledAt": "2026-08-19T09:12:00.000Z"
}
```

`challenge`/`hearAbout`/`teamSize` come from the booking form popup
questions (`payload.responses`); the Company answer is stored in the
shared `company` column instead of `details`.

`cancelledAt` is null/absent until `BOOKING_CANCELLED` arrives.

## 5. Design Notes

- **`type` is server-set**: there is no generic public create route — the
  public POST endpoints hardcode the type (contact/pilot) and the webhook
  hardcodes `CALL`. The enum exists so one table holds all three.
- **`details` is a free-form `Json` column** (default `{}`), shaped and
  validated per type in the application layer (zod), the established
  pattern. The DB stores one row per lead regardless of type; the type
  determines which shape `details` must match.
- **No relational tables** — leads reference nothing (no client table yet)
  and nothing references them. No FKs, no cascade concerns.
- **Indexes**: `@@index([type, status])` serves the admin list filters;
  `@@index([createdAt])` serves the default newest-first sort. `calBookingUid`
  lives inside `details` JSON — lookups by it use a JSON path filter
  (`details: { path: ['calBookingUid'], equals }`), matching the
  per-locale slug pattern already used for content. At V1 lead volume this
  is fine; if it ever becomes hot, promote `calBookingUid` to a real column
  with a unique index.
- **Status is a column, not JSON** — it is queried, filtered, and indexed.
- **`updatedAt`** is the status-change timestamp for the admin UI.

## 6. Product Rule → Enforcement Map

| Rule | Enforcement |
|---|---|
| Type is server-set | No generic create route; type hardcoded per endpoint (app layer) |
| Status starts `NEW` | `@default(NEW)` + explicit default in service layer |
| Cancel closes matched CALL lead | App layer: JSON path lookup on `calBookingUid` → update status |
| Reschedule updates booking info | App layer: same lookup → update `details` |
| Unknown UID on cancel/reschedule | App layer: no-op with log |
| Idempotent webhook creates | App layer: existence check on `calBookingUid` before insert |
| Popup answers map onto the lead (Company → column, challenge/hearAbout/teamSize → details) | App layer: webhook `responses` mapping, missing keys omitted |
| Honeypot never persists | App layer: reject before service/repository |
| Admin edits status only | PATCH route accepts only `status` (zod), no other fields |
| Immutable submission data | No update route for `details`/`fullName`/`workEmail`/`company` |

## 7. Migration Plan

```bash
pnpm exec prisma migrate dev --name add_lead_model
```

(Generates the migration and applies it locally. Production applies the
committed migration via `prisma migrate deploy` against Neon before the API
merge, per `03-Engineering/Deployment.md`.)
