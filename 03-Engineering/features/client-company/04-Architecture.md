# BlihOps V1 Client and Company Feature — Architecture

## 1. Purpose

This document describes the runtime flows, application boundaries, access
checks, scheduled behavior, and frontend/API responsibilities for the Client and
Company feature.

## 2. Component Boundaries

```text
blihops-web
├── Public Website
├── Shared Authentication
└── Client Workspace experience
        │
        ▼
 blihops-api
 ├── Auth / Better Auth
 ├── Client and Company domain
 ├── Talent discovery
 ├── Pods
 ├── Interview Requests
 ├── Pilot Requests and Pilots
 ├── Activity / Audit
 └── Scheduled jobs
        │
        ▼
 PostgreSQL + protected file/storage services

blihops-admin ───────────────► blihops-api Admin endpoints
```

The Client Workspace is not a separate backend application or database model.
The API creates a Company scope from the authenticated Client Account.

## 3. Lead Conversion and Invitation Flow

```text
Admin submits Company conversion
→ API validates Lead and duplicate Company policy
→ Transaction begins
→ Create Company(PILOT_PREPARATION)
→ Create Better Auth User(role=CLIENT)
→ Create ClientAccount(PENDING_INVITATION)
→ Create invitation lifecycle record
→ Mark Lead CONVERTED
→ Commit transaction
→ Issue Better Auth invitation/reset token
→ Send invitation email
→ Return Company Detail
```

The email is sent after the database transaction commits. A delivery failure
leaves the Client Account pending and gives Admin a resend action; it does not
roll back the Company or create duplicates.

## 4. Invitation Acceptance Flow

```text
Client opens emailed link
→ Web renders /accept-invitation
→ API/Better Auth validates token
→ Client sets password
→ Token is consumed
→ ClientAccount becomes ACTIVE
→ Invitation becomes ACCEPTED
→ Session is created
→ Web redirects to /client-workspace
```

The redirect destination is role-based. The Client never supplies a Company ID
as an authorization decision.

## 5. Client Workspace Request Flow

```text
Client opens /client-workspace
→ Web requests /api/v1/client/work
→ API reads session User ID
→ API loads ClientAccount
→ API resolves companyId
→ API checks ClientAccount ACTIVE
→ API checks Company not ARCHIVED
→ API queries Company-scoped projection
→ Web renders Work, Talent, Pod, request, or Pilot state
```

All Client repositories require a server-derived `companyId`. Route parameters
and request bodies cannot widen the scope.

## 6. Company Access Enforcement

Authorization is enforced in layers:

1. Authentication middleware validates the session.
2. Role middleware requires `CLIENT` or `ADMIN` as appropriate.
3. Client context middleware loads `ClientAccount` and Company.
4. Status guard rejects deactivated accounts and archived Companies.
5. Repository/service methods include the resolved Company ID in every query.
6. Response serializers remove Admin-only and private Pod fields.

A deactivated Client loses existing sessions. An archived Company blocks Client
Workspace requests even if the Client Account remains active.

## 7. Client Talent and Pod Flow

### Talent discovery

```text
Client requests directory
→ API filters visible, active Talent
→ API removes internal fields
→ Web renders cards
→ Client opens detail
→ Client adds Talent to private Pod or requests interview
```

### Private Pod mutation

```text
Client creates/edits Pod
→ API verifies Company ownership
→ API validates visible Talent membership
→ API validates optional Pod Lead membership
→ Transaction saves Pod and members
→ No Admin Pod event is exposed
```

### Pod interview action

```text
Client selects Request Interviews for Pod
→ API loads current Pod members
→ API creates one InterviewRequestGroup
→ API creates one InterviewRequest per unique member
→ API returns individual request summaries
→ Admin sees individual requests only
```

### Pod Pilot selection

```text
Client selects Pod in Pilot Request
→ API loads current Pod members
→ API expands members into request participant rows
→ API removes duplicates
→ API stores source metadata internally
→ Later Pod changes do not modify the request
```

## 8. Interview Request Flow

```text
Client submits individual or grouped request
→ Request is PENDING
→ Admin checks Talent availability manually
→ Admin contacts Talent outside platform
→ Admin schedules or rejects request
→ Client sees safe status/schedule projection
→ Interview occurs outside platform
→ Admin marks COMPLETED or CANCELLED
```

The API treats each Talent request independently even when the Client created
several through one Pod action.

## 9. Pilot Request Flow

```text
Client opens Work while Company=PILOT_PREPARATION
→ Web displays Pilot Request form
→ Client submits goals, measurements, date, context, and participants
→ API validates eligibility and participant snapshot
→ PilotRequest=SUBMITTED
→ Admin opens Company Detail
→ Admin marks IN_REVIEW
→ [Reject] → request remains editable/resubmittable
→ [Approve] → create Pilot=PLANNED
→ Admin confirms operational details
→ Admin starts Pilot
→ Pilot=ACTIVE; Company=ON_PILOT
```

A Client cannot submit a duplicate non-final request or submit after a successful
Pilot has completed.

## 10. Pilot Execution and Completion Job

When Admin starts a Pilot:

```text
Validate Pilot=PLANNED
→ Set actual startDate
→ Set endDate=startDate + 14 days
→ Set Pilot=ACTIVE
→ Set Company=ON_PILOT
→ Write activity records
```

A scheduled job runs regularly:

```text
Find ACTIVE Pilots where endDate <= now
→ Transaction locks Pilot and Company
→ Set Pilot=COMPLETED and completedAt
→ Set Company=PILOT_COMPLETED
→ Write activity records
→ Do not send general notification
```

If Admin cancels a planned or active Pilot:

```text
Set Pilot=CANCELLED
→ Record reason and actor
→ Company returns to PILOT_PREPARATION
→ Successful Pilot eligibility remains available
```

Only `COMPLETED` consumes the Company's one free Pilot opportunity.

## 11. Progress Update Flow

```text
Client opens ACTIVE Pilot
→ Client submits text
→ API verifies Pilot belongs to Client Company and is ACTIVE
→ API assigns author and createdAt
→ API creates immutable update
→ Client and Admin projections show it
```

There is no update endpoint. Admin removal uses soft removal and creates an audit
event. Removed updates disappear from the Client projection.

## 12. Admin Company Detail Flow

Company Detail loads a composed Admin projection:

```text
Company profile
+ Client Account/invitation
+ derived access state
+ Interview Requests
+ Pilot Requests
+ Pilot history
+ Notes
+ Activity
```

The projection does not include private Pod records. It may include a safe
indication that an Interview Request was generated from a grouped Client action,
but not Pod title, membership, leader, or description.

Admin actions use explicit transition commands rather than arbitrary status
patches where possible:

- Review/reject/approve Pilot Request
- Start/cancel Pilot
- Mark Company Active/Archived
- Deactivate/reactivate Client Account
- Resend/replace invitation

## 13. Scheduled and Retryable Work

- Pilot completion is handled by an idempotent scheduled job.
- Invitation email delivery is retryable from Company Detail.
- Repeated webhook/request delivery must not duplicate a conversion, invitation,
  Pilot participant, or Interview Request group.
- All status jobs use transactions and re-check the current status before
  mutating data.

## 14. Activity and Audit

The internal audit stream records actor, action, resource, previous state, next
state, and timestamp for important mutations.

The Client activity projection is separate and includes only client-safe events.
It excludes Pod mutations, internal notes, private audit information, and
technical events.

## 15. Frontend Route Responsibilities

### blihops-web

```text
/[locale]/auth/accept-invitation
/[locale]/client-workspace
/[locale]/client-workspace/talents
/[locale]/client-workspace/talents/:talentProfileId
/[locale]/client-workspace/pods
/[locale]/client-workspace/pods/:podId
/[locale]/client-workspace/interview-requests
```

The first implementation may use a dialog for Talent Detail, but the domain
API should support a direct detail route.

### blihops-admin

```text
/(dashboard)/companies
/(dashboard)/companies/:companyId
```

Interview Requests, Pilot Requests, Pilots, notes, and activity remain nested in
Company Detail rather than becoming separate top-level Admin navigation items.

## 16. Failure and Edge States

- Invitation email fails: account remains pending; Admin can resend.
- Invitation expires: replace invitation; do not create another Client Account.
- Client deactivated during Pilot: revoke access; preserve Pilot state/history.
- Company archived during Pilot: block Client access; preserve Pilot history.
- Talent becomes unavailable: show unavailable participant placeholder and let
  Admin adjust planned participants.
- Pilot cancelled: preserve history, return Company to preparation, keep quota.
- Pilot completion job retries: status check prevents duplicate completion events.
- Client submits after eligibility changes: API rejects with a safe status error.

## 17. Out of Scope / Accepted V1 Debt

- Multiple Client Accounts per Company
- Separate Workspace entity
- Contract and project-management modules
- General notifications and email updates
- Automatic talent availability confirmation
- Direct Client-to-Talent communication
- Milestones, tasks, timesheets, and attachments on progress updates
- Fully automated transition from completed Pilot to Active partnership
