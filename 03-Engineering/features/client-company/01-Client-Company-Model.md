# BlihOps V1 Client and Company Feature — Domain Model

## 1. Purpose

This document is the source of truth for the Client account, Company, logical
Client Workspace, Interview Request, Pod, Pilot Request, Pilot, and progress
update behavior.

It defines business rules and state transitions. Prisma details belong in
`02-Data-Model.md`; HTTP contracts belong in `03-API-Design.md`; request flows
and enforcement belong in `04-Architecture.md`.

## 2. Scope

V1 supports:

- Lead conversion into a Company and invited Client Account
- One primary Client Account per Company
- One Company-scoped Client Workspace experience
- Private Client Pods
- Optional individual or Pod-based Interview Requests
- One successful free 14-day Pilot opportunity per Company
- Admin-managed Pilot execution and Client-submitted progress updates
- Active partnership and archived Company states

V1 does not create a separate `ClientWorkspace` database model. The Company ID
is the data and authorization scope for the Client Workspace experience.

## 3. Core Relationships

```text
User (role=CLIENT)
        │
        ▼
ClientAccount ───────────────► Company
                                  │
                                  ├── PilotRequests
                                  ├── Pilots
                                  ├── InterviewRequests
                                  └── private ClientPods
```

V1 enforces one active or pending Client Account per Company, while keeping the
Company independent from the User record for future representative replacement.

## 4. Client Account Lifecycle

```text
PENDING_INVITATION → ACTIVE → DEACTIVATED
```

### PENDING_INVITATION

The User and ClientAccount exist, but the Client has not activated the account.
The invitation is single-use and expires. An expired invitation can be replaced
by Admin without creating another Client Account.

### ACTIVE

The Client has set a password and may enter the Company-scoped Workspace if the
Company is not archived.

### DEACTIVATED

Admin has revoked access. Existing sessions are invalidated, new login is
blocked, and all Client API requests fail authorization. Reactivation is a
separate Admin action.

## 5. Company Lifecycle

```text
PILOT_PREPARATION → ON_PILOT → PILOT_COMPLETED → ACTIVE
                                                ↘ ARCHIVED
```

`ARCHIVED` can be reached from any non-archived state. Restoring a Company does
not reactivate a deactivated Client Account.

| Status | Entry condition | Client behavior |
|---|---|---|
| `PILOT_PREPARATION` | Client Account is activated or a Pilot is cancelled | Can prepare and submit the free Pilot Request |
| `ON_PILOT` | Admin starts the Pilot | Can view Pilot and submit updates |
| `PILOT_COMPLETED` | Active Pilot reaches its 14-day end | Can view read-only Pilot history; no new Pilot Request |
| `ACTIVE` | Admin confirms ongoing relationship | Can view partnership state and Pilot history; no Pilot Request |
| `ARCHIVED` | Admin closes relationship or blocks Company | Workspace access is denied; history is preserved |

## 6. Lead Conversion Flow

```text
Admin qualifies Lead
→ Admin opens conversion form
→ System checks duplicate Company rules
→ System creates Company
→ System creates Client User with role CLIENT
→ System creates ClientAccount(PENDING_INVITATION)
→ System creates invitation lifecycle record
→ System sends invitation email
→ System marks Lead CONVERTED
```

Company creation, Client Account creation, and Lead conversion are one logical
operation. Invitation delivery can be retried after the records commit; a mail
provider failure must not create duplicate Company or User records.

The initial Company status is `PILOT_PREPARATION` and the initial Client Account
status is `PENDING_INVITATION`.

## 7. Workspace Resolution and Authorization

The client route is `/client-workspace`; it does not accept a client-controlled
Workspace ID.

```text
Session User ID
→ ClientAccount.userId
→ ClientAccount.companyId
→ Company-scoped query
```

Every Client request must verify:

```text
User.role = CLIENT
ClientAccount.status = ACTIVE
Company.status != ARCHIVED
ClientAccount.companyId = resolved Company ID
```

A route or request body must never be trusted as the source of Client Company
scope. If a Company ID is accepted internally, it must match the ClientAccount
Company ID.

## 8. Pilot Request Rules

Only a Company in `PILOT_PREPARATION` may submit a Client Pilot Request.

The request contains:

- Title
- Business goals
- Success measurements
- Expected start date
- Fixed duration of 14 days
- Optional additional context
- Individual Talent selections
- Pod selections expanded into individual participant snapshots

Interview completion is recommended but not required. Any visible Talent may be
included.

A Company can have only one non-final request or Pilot in progress. Rejected
requests may be edited and resubmitted. A cancelled Pilot does not consume the
free Pilot opportunity and returns the Company to `PILOT_PREPARATION`.

## 9. Pilot Request Lifecycle

```text
SUBMITTED
    ↓
IN_REVIEW
    ├── REJECTED → SUBMITTED after revision
    └── APPROVED → creates PLANNED Pilot
```

Approval does not start the Pilot. Admin must explicitly start it.

## 10. Pilot Lifecycle

```text
PLANNED → ACTIVE → COMPLETED
    ↘              ↘
      CANCELLED      CANCELLED
```

- `PLANNED`: approved request, preparation not complete.
- `ACTIVE`: Admin started the fixed 14-day Pilot.
- `COMPLETED`: end date reached; this is the only state that consumes the free
  Pilot opportunity.
- `CANCELLED`: Pilot will not continue; quota remains available.

A scheduled job transitions an active Pilot to `COMPLETED` at its end date. It
also moves the Company from `ON_PILOT` to `PILOT_COMPLETED`.

## 11. Interview Requests

Interview Requests are optional and are not a Pilot gate.

A Client may request:

- One individual Talent interview
- Interviews for all members currently in a private Pod

A Pod action creates individual Talent requests grouped by one client action.
Each request has its own scheduling and outcome state. Admin sees individual
Talent requests, not the private Pod structure.

Recommended statuses:

```text
PENDING
SCHEDULED
COMPLETED
REJECTED
CANCELLED
```

Blihops owns availability checks, scheduling, and direct Talent communication.
Completing an interview does not create a Pilot or participant assignment.

## 12. Private Pod Rules

Pods belong to a Company and are visible only to that Company's Client Account.
Admin cannot list, inspect, edit, or expose Pod content.

Client actions:

- Create, rename, describe, and archive Pods
- Add or remove visible Talent
- Assign one current member as Pod Lead
- Request interviews for all members
- Use a Pod as a Pilot Request source

Pod membership does not reserve, notify, contact, assign, or add Talent to a
Pilot. Removing a Pod Lead clears the lead. A Pod-to-request operation snapshots
current members; later Pod changes do not update existing operations.

## 13. Active Pilot and Progress Updates

Admin owns all official Pilot information:

- Title
- Goals
- Success measurements
- Start and end dates
- Participants
- Pilot status

Client permissions during `ACTIVE`:

- Read official Pilot information
- Submit progress updates
- Read updates and participants

A Progress Update contains only immutable text, author, and created timestamp.
The Client cannot edit or delete it. Admin can remove it after confirmation;
removal remains auditable.

V1 has no milestones.

## 14. Completed and Active Partnership

After completion:

```text
Pilot: ACTIVE → COMPLETED
Company: ON_PILOT → PILOT_COMPLETED
```

The Client can view the completed Pilot read-only and cannot submit new updates
or a new Pilot Request.

Admin then chooses:

```text
PILOT_COMPLETED → ACTIVE
PILOT_COMPLETED → ARCHIVED
```

`ACTIVE` is a simple partnership state in V1. It exposes Pilot history and basic
partnership statistics only; it does not introduce contract or project
management functionality.

## 15. Audit Requirements

Record at minimum:

- Company creation, conversion, edit, archive, and restore
- Client invitation sent, resent, replaced, accepted, and expired
- Client Account activation, deactivation, and reactivation
- Pilot Request submission, review, rejection, resubmission, and approval
- Pilot creation, start, cancellation, completion, and outcome
- Interview Request creation and status changes
- Progress Update creation and Admin removal

Private Pod mutations may be retained for security/audit purposes, but must not
be exposed in the Admin product as Company operational data.

## 16. Out of Scope

- Multiple Client Accounts per Company
- A Workspace database entity
- Multiple successful free Pilots
- Contracts, invoices, billing, payments, or signatures
- Daily tasks, chat, time tracking, or delivery management
- General notification center or automated operational email
