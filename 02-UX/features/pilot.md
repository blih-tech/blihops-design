# BlihOps V1 Pilot Feature

## 1. Purpose

A Pilot is a single free 14-day trial opportunity that lets a Client evaluate
working with Blihops and selected Talent before deciding whether to enter an
ongoing relationship.

A Pilot Request submitted from the public website creates a Lead. A Pilot
Request submitted from the authenticated Client Workspace is a different,
Company-scoped operational request. This document focuses on the authenticated
Client flow and the Admin-managed Pilot.

Daily work, contracts, billing, payments, and direct communication remain
outside the platform in V1.

## 2. Related Concepts

| Concept | Meaning |
|---|---|
| Public Pilot form | Public marketing form that creates a `PILOT` Lead |
| Client Pilot Request | Authenticated Client request for the one free 14-day Pilot |
| Talent Pod | Private Client shortlist used to organize visible Talent |
| Interview Request | Optional Client request for Blihops to coordinate an interview |
| Pilot | Admin-approved, planned, active, completed, or cancelled trial |
| Pilot Participant | Talent explicitly confirmed by Admin for a Pilot |
| Progress Update | Permanent Client-submitted note during an active Pilot |

A Client Pilot Request does not start a Pilot automatically. Pod membership and
Interview completion do not create Pilot participation automatically.

## 3. Actors

### Client

- Browses visible Talent.
- Organizes private Pods.
- Optionally requests individual or Pod-based interviews.
- Submits and, after rejection, revises a Pilot Request.
- Views Pilot information.
- Submits permanent progress updates during an active Pilot.

### Admin

- Reviews Pilot Requests.
- Confirms the intended Pilot details and participants.
- Starts and manages the Pilot.
- Updates official goals, measurements, dates, participants, and status.
- Removes Client progress updates when necessary.
- Decides whether a completed Pilot becomes Active or Archived.

### Talent

- Is coordinated by Blihops outside the platform.
- Has no Pilot-management access in V1.

## 4. Pilot Eligibility

A Company may submit a Client Pilot Request only when its relationship status is
`PILOT_PREPARATION`.

The Pilot feature is unavailable when the Company is:

```text
ON_PILOT
PILOT_COMPLETED
ACTIVE
ARCHIVED
```

A Company has one successful free Pilot opportunity. Only a Pilot that reaches
`COMPLETED` consumes it.

- Rejected requests do not consume the opportunity.
- Cancelled Pilots do not consume the opportunity.
- A rejected request may be edited and resubmitted.
- A cancelled Pilot returns the Company to `PILOT_PREPARATION`.
- Only one non-final Pilot Request or Pilot may be in progress at a time.

## 5. Client Pilot Request

### 5.1 Required and Optional Fields

| Field | Required | Notes |
|---|---|---|
| Pilot title | Yes | Client-provided title |
| Business goals | Yes | What the Client wants to achieve |
| Success measurements | Yes | How the Client expects success to be evaluated |
| Expected start date | Yes | Requested date; Admin confirms the actual date |
| Expected duration | Automatic | Fixed at 14 days; not client-editable |
| Additional context | No | Free-text context for Admin |
| Individual Talent participants | No | Any visible Talent may be selected |
| Pod participant sources | No | Pod members expand into individual participants |

The Client may include any visible Talent. Interview completion is recommended
but is not required.

### 5.2 Participant Expansion

When a Client selects a Pod, the system copies the Pod's current members into
the Pilot Request as individual participants. A Talent selected both directly and
through a Pod appears only once.

The Admin sees the individual participants, not the private Pod title,
description, leader, or membership structure.

The participant list is a snapshot. Later Pod changes do not alter a submitted
request or an active Pilot.

## 6. Pilot Request Lifecycle

```text
SUBMITTED
    ↓
IN_REVIEW
    ├── REJECTED → Client edits and resubmits
    └── APPROVED → Pilot becomes PLANNED
```

| Status | Meaning |
|---|---|
| `SUBMITTED` | Client has sent the request to Admin |
| `IN_REVIEW` | Admin is evaluating the request |
| `REJECTED` | Admin did not approve it; Client may revise and resubmit |
| `APPROVED` | Admin accepted it and a planned Pilot exists |

An approval does not start the Pilot automatically.

## 7. Pilot Lifecycle

```text
PLANNED → ACTIVE → COMPLETED
    ↘              ↘
      CANCELLED      CANCELLED
```

| Status | Meaning |
|---|---|
| `PLANNED` | Request approved; dates and participants are being prepared |
| `ACTIVE` | Admin explicitly started the 14-day Pilot |
| `COMPLETED` | The Pilot reached its end and counts as the successful free Pilot |
| `CANCELLED` | The Pilot will not continue; free Pilot opportunity remains available |

The expected duration is fixed at 14 days. The actual end date is calculated
from the actual start date. A scheduled job completes an active Pilot when its
end date is reached.

## 8. Company Transitions

| Event | Company status |
|---|---|
| Client activates account | `PILOT_PREPARATION` |
| Client submits request | `PILOT_PREPARATION` |
| Request is rejected | `PILOT_PREPARATION` |
| Request is approved | `PILOT_PREPARATION` |
| Admin starts Pilot | `ON_PILOT` |
| Pilot is cancelled | `PILOT_PREPARATION` |
| Pilot completes | `PILOT_COMPLETED` |
| Relationship continues | `ACTIVE` |
| Relationship ends | `ARCHIVED` |

After `PILOT_COMPLETED`, the Client can view the completed Pilot but cannot
submit another Pilot Request.

## 9. Admin Management

Within Company Detail, Admin can:

- Review request fields and participants
- Move a request to In Review
- Reject with a reason
- Approve a request
- Confirm actual dates and participants before starting
- Start the Pilot
- Edit official goals, measurements, dates, and participants
- Cancel the Pilot
- View and remove Client progress updates
- Complete or review the Pilot outcome
- Set the Company to Active or Archived after completion

Admin may adjust the operational Pilot details before or during execution. The
Client's submitted request remains an auditable record.

## 10. Active Pilot Work Page

The Client sees:

- Pilot title
- Active status
- Start date
- End date
- Days remaining
- Business goals
- Success measurements
- Participants
- Progress update timeline
- Add Progress Update action

The Client cannot change official Pilot information or participants.

## 11. Progress Updates

A Client update contains only:

- Update text
- Author, generated automatically
- Created date/time, generated automatically

Updates are immutable. The Client cannot edit or delete an update. Admin may
remove an update after confirmation; the removal remains traceable in internal
audit history.

There are no milestones in V1.

## 12. Completed Pilot Experience

When the Pilot completes:

```text
Pilot: ACTIVE → COMPLETED
Company: ON_PILOT → PILOT_COMPLETED
```

The Client remains able to log in and view:

- Completed Pilot card
- Goals and measurements
- Dates
- Participants
- Progress updates
- Pilot history

The completed Pilot is read-only. The Work page may show a message encouraging
the Client to begin an ongoing relationship with Blihops.

## 13. Active Partnership Experience

When Admin changes the Company to `ACTIVE`, the Client sees:

- Active partnership status
- Partnership start date
- Simple relationship statistics
- Completed Pilot history

V1 does not include contracts, tasks, daily work management, time tracking,
chat, billing, or payments.

## 14. Out of Scope

- Multiple successful free Pilots for one Company
- Automatic Pilot approval or start
- Talent matching or automatic availability confirmation
- Direct Client-to-Talent messaging
- Video interview hosting
- Milestones and task management
- Contracts and electronic signatures
- Payments and invoicing
- Automatic conversion from completed Pilot to Active partnership
- General notifications or notification center
