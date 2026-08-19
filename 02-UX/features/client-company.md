# BlihOps V1 Client and Company Feature

## 1. Purpose

This document defines the Client account, Company relationship, invitation,
access, and lifecycle experience for BlihOps V1.

A Company is the business record and data boundary. The Client Workspace is a
client-facing web experience scoped to the Company; it is not a separate
Workspace database entity in V1.

## 2. Core Concepts

| Concept | Meaning |
|---|---|
| User | Better Auth identity with role `ADMIN`, `CLIENT`, or `TALENT` |
| Client Account | The relationship between one Client User and one Company; owns account access status |
| Company | The business relationship, Pilot history, requests, and client data boundary |
| Client Workspace | The protected Web experience rendered from the authenticated Client's Company context |
| Pilot Request | A Client-submitted request for the one free 14-day Pilot opportunity |
| Pilot | An Admin-approved, planned, active, completed, or cancelled trial |
| Client Pod | A private Client planning group of visible Talent |

V1 supports one primary Client Account per Company. The data model should not
make the Company dependent on the continued existence of that User so the
representative can be replaced later.

## 3. Actors

### Admin

- Converts qualified Leads into Companies.
- Creates the Client Account and sends the invitation.
- Manages Company and Client Account access.
- Reviews Interview Requests and Pilot Requests.
- Starts, manages, completes, or cancels Pilots.
- Decides whether a completed Pilot relationship becomes Active or Archived.

### Client

- Activates the invited account.
- Enters only the Workspace for their Company.
- Browses visible Talent.
- Creates private Pods.
- Optionally requests interviews.
- Submits one Pilot opportunity request, subject to Company status.
- Views Pilot information and submits permanent progress updates during an
  active Pilot.

## 4. Account and Company Statuses

### 4.1 Client Account Status

```text
PENDING_INVITATION → ACTIVE → DEACTIVATED
```

| Status | Meaning |
|---|---|
| `PENDING_INVITATION` | Client User exists but has not activated the invitation |
| `ACTIVE` | Client can authenticate and access the Company experience when the Company is available |
| `DEACTIVATED` | Admin has revoked access; login and Workspace requests are blocked |

Invitation expiry is tracked by the invitation record, not as a permanent
Client Account status. An expired invitation remains pending until it is
replaced or the account is deactivated.

### 4.2 Company Relationship Status

```text
PILOT_PREPARATION → ON_PILOT → PILOT_COMPLETED → ACTIVE
                                                ↘ ARCHIVED
```

`ARCHIVED` may be reached from any non-archived Company state.

| Status | Meaning |
|---|---|
| `PILOT_PREPARATION` | Client has access and may prepare for or request the free Pilot |
| `ON_PILOT` | Admin has started the 14-day Pilot |
| `PILOT_COMPLETED` | The one free Pilot completed successfully; continuation decision is pending |
| `ACTIVE` | Admin and Client agreed to continue the relationship |
| `ARCHIVED` | Relationship is closed or unavailable; history is preserved and access is blocked |

## 5. Lead Conversion and Invitation Flow

```text
Admin opens a qualified Lead
→ Admin confirms Company and Client representative details
→ System creates Company
→ System creates Client User with role CLIENT
→ System creates Client Account as PENDING_INVITATION
→ System creates the Company-scoped client experience context
→ System sends the account invitation email
→ Lead becomes CONVERTED
→ Admin opens Company Detail
```

There is no `ClientWorkspace` database record. The client experience is resolved
from the authenticated User:

```text
Authenticated User
→ Client Account
→ Company ID
→ Company-scoped Client Workspace
```

The Company starts as `PILOT_PREPARATION`. The Client Account starts as
`PENDING_INVITATION`.

The invitation form requires:

- Client full name
- Client work email
- Company name
- Company profile information required by Admin

The Client sets the password through the invitation link. Admin never sets or
receives the Client's password.

## 6. Client Activation and Redirect

```text
Client opens invitation link
→ System validates single-use, unexpired invitation
→ Client creates password
→ Client Account becomes ACTIVE
→ Invitation becomes ACCEPTED
→ Client is redirected to /client-workspace
→ API resolves Company ID from Client Account
```

The Client does not choose a Company or Workspace ID. A Client cannot switch to
another Company by changing a URL parameter.

## 7. Access Rules

Client Workspace access requires all of the following:

```text
Authenticated User role is CLIENT
Client Account status is ACTIVE
Company status is not ARCHIVED
Client Account belongs to the resolved Company
```

### Deactivated Client Account

- Existing sessions are revoked.
- New login is blocked.
- Client API requests are rejected.
- Company and Pilot history remain intact.
- Reactivation is a separate Admin action.

### Archived Company

- Company history is preserved.
- Client Workspace requests are blocked.
- The Client Account status is not changed automatically.
- Restoring the Company does not reactivate a deactivated Client Account.
- If the Client Account is active, restoring the Company may restore Workspace
  access.

The UI should explain the reason for blocked access: invitation pending, Client
account deactivated, or Company archived.

## 8. Pilot Eligibility Gate

Only a Company in `PILOT_PREPARATION` can submit a Pilot Request.

Pilot Requests are unavailable when the Company is:

```text
ON_PILOT
PILOT_COMPLETED
ACTIVE
ARCHIVED
```

A rejected Pilot Request does not consume the free Pilot opportunity. A Pilot
cancelled before completion also does not consume it and returns the Company to
`PILOT_PREPARATION`.

A completed Pilot consumes the one free Pilot opportunity. After completion, the
Company moves to `PILOT_COMPLETED`, then Admin chooses `ACTIVE` or `ARCHIVED`.

## 9. Admin Company Experience

### Companies list

The list displays separate account and relationship information:

- Company name
- Client name and email
- Client Account status
- Company relationship status
- Workspace access state
- Current or latest Pilot state
- Updated date

### Company Detail

Company Detail includes:

- Company profile
- Client Account and invitation status
- Workspace access reason
- Interview Requests
- Pilot Requests and Pilot history
- Internal notes
- Internal activity

Admin actions include:

- Edit Company information
- Resend or replace invitation
- Deactivate or reactivate Client Account
- Archive or restore Company
- Review and decide on Pilot Requests
- Start, manage, complete, or cancel Pilots
- Choose Active or Archived after a completed Pilot

Client Pods are not visible or manageable from Company Detail.

## 10. Privacy Boundaries

Pods are completely private to the Client. Admin cannot see:

- Pod title
- Pod description
- Pod Leader
- Pod membership
- Pod history
- How the Client organizes or compares Talent

When a Client requests interviews from a Pod, Admin sees only the resulting
individual Talent Interview Requests. When a Client includes a Pod in a Pilot
Request, Admin sees only the resulting individual Pilot participants.

The operational record may retain an internal source reference to support
history and grouping, but the Admin product surface must not expose private Pod
content.

## 11. Out of Scope

- Public Client registration
- Multiple Client Accounts per Company in V1
- Client team management
- Contract, billing, payment, or invoice management
- Client-to-Talent messaging
- Separate Workspace entity or multiple Workspaces per Company
- Automated transition from Pilot completion to Active
- General notifications or a notification center
