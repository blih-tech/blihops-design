# BlihOps V1 Client and Company Feature — API Design

## 1. Purpose

This document defines the planned API surface for Client Accounts, Companies,
Client Workspace data, private Pods, Interview Requests, Pilot Requests, Pilots,
and progress updates.

The API uses the existing `/api/v1` convention and enforces role, ownership,
status, and Company scope on the server.

## 2. Authorization Context

### Admin

Admin endpoints require the existing Admin role middleware. Admin may access
authorized Companies, Talent, Interview Requests, Pilots, and audit data.

### Client

Client endpoints derive Company scope from the authenticated session:

```text
session.user.id
→ ClientAccount.userId
→ ClientAccount.companyId
```

The Client does not submit a Company ID as the source of authorization. If a
Company ID is accepted for an internal lookup, it must equal the resolved ID.

Client authorization requires:

```text
role = CLIENT
ClientAccount.status = ACTIVE
Company.status != ARCHIVED
```

### No Workspace ID

There is no `/workspaces/:workspaceId` resource in V1. The Client Workspace is
represented by Company-scoped endpoints.

## 3. Route Inventory

### 3.1 Admin Companies and Client Accounts

```text
GET    /api/v1/admin/companies
POST   /api/v1/admin/companies
GET    /api/v1/admin/companies/:companyId
PATCH  /api/v1/admin/companies/:companyId
POST   /api/v1/admin/companies/:companyId/invitation
POST   /api/v1/admin/companies/:companyId/deactivate-client
POST   /api/v1/admin/companies/:companyId/reactivate-client
POST   /api/v1/admin/companies/:companyId/archive
POST   /api/v1/admin/companies/:companyId/restore
```

`POST /admin/companies` supports manual creation and conversion from a qualified
Lead. Company creation with a Client representative creates the Client Account
and invitation in one logical transaction.

The list response includes separate:

- Company relationship status
- Client Account status
- Invitation status
- Derived Workspace access state

The Admin API never returns Client Pod records.

### 3.2 Client Workspace Context

```text
GET /api/v1/client/me
GET /api/v1/client/work
```

`/client/me` returns the authenticated Client's safe identity and Company
summary. `/client/work` returns the Work page projection based on Company status:
Pilot preparation, request review, planned/active Pilot, completed history, or
Active partnership.

### 3.3 Client Talent Directory

```text
GET /api/v1/client/talents
GET /api/v1/client/talents/:talentProfileId
```

Only active, visible, client-eligible Talent fields are returned. Internal rates,
assessments, notes, and compensation are excluded at the serializer boundary.

### 3.4 Client Private Pods

```text
GET    /api/v1/client/pods
POST   /api/v1/client/pods
GET    /api/v1/client/pods/:podId
PATCH  /api/v1/client/pods/:podId
POST   /api/v1/client/pods/:podId/members
DELETE /api/v1/client/pods/:podId/members/:talentProfileId
POST   /api/v1/client/pods/:podId/interview-requests
POST   /api/v1/client/pods/:podId/archive
```

All Pod operations verify that the Pod belongs to the session Company. Pod
responses are never available to Admin Company endpoints.

A Pod interview action expands the current membership into individual Interview
Requests and returns the created request group plus individual request summaries.

### 3.5 Client Interview Requests

```text
GET  /api/v1/client/interview-requests
POST /api/v1/client/interview-requests
GET  /api/v1/client/interview-requests/:requestId
```

The Client can create an individual request from a Talent Profile. A Pod batch
uses the Pod route and creates one request per member.

The Client cannot directly change status, schedule, add Admin notes, or contact
Talent. Client responses expose only client-safe status, schedule, and message
fields.

### 3.6 Client Pilot Requests

```text
GET   /api/v1/client/pilot-requests
POST  /api/v1/client/pilot-requests
GET   /api/v1/client/pilot-requests/:requestId
PATCH /api/v1/client/pilot-requests/:requestId
POST  /api/v1/client/pilot-requests/:requestId/resubmit
```

Creation is permitted only when:

```text
Company.status = PILOT_PREPARATION
No non-final request or Pilot exists
No completed Pilot exists
```

The server sets `durationDays = 14`. The Client cannot set or change duration.

`PATCH` is available for a rejected request. Resubmission moves the request back
to `SUBMITTED` and increments the resubmission count.

Participant payloads contain individual Talent IDs and optional Pod IDs. The
service expands Pod members, removes duplicates, validates visibility, and stores
a snapshot.

### 3.7 Client Pilot and Progress Updates

```text
GET  /api/v1/client/pilots
GET  /api/v1/client/pilots/:pilotId
POST /api/v1/client/pilots/:pilotId/updates
GET  /api/v1/client/pilots/:pilotId/updates
```

The Client can create a progress update only while the Pilot is `ACTIVE`. The
payload is only:

```json
{
  "text": "The first integration is ready for testing."
}
```

The server assigns author and creation time. There is no Client update or delete
endpoint.

### 3.8 Admin Company Operations

```text
GET    /api/v1/admin/companies/:companyId/interview-requests
GET    /api/v1/admin/companies/:companyId/pilot-requests
GET    /api/v1/admin/companies/:companyId/pilots
GET    /api/v1/admin/companies/:companyId/activity
POST   /api/v1/admin/companies/:companyId/notes
```

### 3.9 Admin Interview Requests

```text
GET   /api/v1/admin/interview-requests/:requestId
PATCH /api/v1/admin/interview-requests/:requestId
```

Admin may update:

- Status
- Scheduled date/time
- Time zone
- Client-visible coordination message
- Internal notes
- Rejection/cancellation reason

The Admin API may show that a request came from a grouped client action, but it
must not expose private Pod title, description, leader, or membership structure.

### 3.10 Admin Pilot Requests and Pilots

```text
GET   /api/v1/admin/pilot-requests/:requestId
POST  /api/v1/admin/pilot-requests/:requestId/in-review
POST  /api/v1/admin/pilot-requests/:requestId/reject
POST  /api/v1/admin/pilot-requests/:requestId/approve

GET   /api/v1/admin/pilots/:pilotId
PATCH /api/v1/admin/pilots/:pilotId
POST  /api/v1/admin/pilots/:pilotId/start
POST  /api/v1/admin/pilots/:pilotId/cancel
POST  /api/v1/admin/pilots/:pilotId/complete
POST  /api/v1/admin/pilots/:pilotId/updates
DELETE /api/v1/admin/pilots/:pilotId/updates/:updateId
POST  /api/v1/admin/companies/:companyId/mark-active
POST  /api/v1/admin/companies/:companyId/mark-archived
```

Approval creates a `PLANNED` Pilot but does not start it. Starting sets actual
start/end dates and moves the Company to `ON_PILOT`. Manual completion is an
Admin operation for exceptional cases; normal completion is scheduled from the
end date.

Admin can manage official Pilot fields and participants. Admin removal of a
progress update is soft removal and audited.

## 4. Payload Shapes

### 4.1 Create Pilot Request

```json
{
  "title": "Customer support automation trial",
  "businessGoals": "Reduce backlog and improve response time.",
  "successMeasurements": "Resolve 80% of priority tickets within one business day.",
  "expectedStartDate": "2026-09-01T00:00:00.000Z",
  "additionalContext": "Initial focus is the German support queue.",
  "individualTalentIds": ["talent_1"],
  "podIds": ["pod_1"]
}
```

The response contains the expanded unique participant list. It does not expose
private Pod details to Admin or other Clients.

### 4.2 Pilot Request Summary

```json
{
  "id": "request_1",
  "status": "IN_REVIEW",
  "title": "Customer support automation trial",
  "businessGoals": "Reduce backlog and improve response time.",
  "successMeasurements": "Resolve 80% of priority tickets within one business day.",
  "expectedStartDate": "2026-09-01T00:00:00.000Z",
  "durationDays": 14,
  "additionalContext": "Initial focus is the German support queue.",
  "participants": [
    {
      "talentProfileId": "talent_1",
      "source": "POD"
    }
  ],
  "submittedAt": "2026-08-20T09:00:00.000Z"
}
```

`source` may be simplified or omitted in Client responses. Admin receives
individual participant information but not private Pod details.

### 4.3 Progress Update

```json
{
  "id": "update_1",
  "text": "The first integration is ready for testing.",
  "author": {
    "id": "user_1",
    "name": "Client Representative"
  },
  "createdAt": "2026-09-04T10:30:00.000Z"
}
```

## 5. Validation Rules

- Title, goals, success measurements, and expected start date are required.
- Duration is always server-controlled as 14 days.
- Additional context has a configured maximum length.
- Selected Talent must be visible and eligible at submission time.
- Selected Pod must belong to the Client Company.
- Duplicate participant Talent IDs are collapsed or rejected consistently.
- A rejected request can be edited and resubmitted.
- A Client cannot submit after Company status leaves `PILOT_PREPARATION`.
- A Client cannot create updates on a completed or cancelled Pilot.
- A Pod Lead must be a current Pod member.
- Interview requests cannot be created for hidden or archived Talent.
- Admin-only fields are rejected from Client payloads, not silently accepted.

## 6. Error Codes

Recommended feature errors:

```text
CLIENT_ACCOUNT_INACTIVE
COMPANY_ARCHIVED
COMPANY_NOT_IN_PILOT_PREPARATION
PILOT_OPPORTUNITY_USED
PILOT_REQUEST_ALREADY_IN_PROGRESS
PILOT_REQUEST_NOT_EDITABLE
PILOT_NOT_ACTIVE
TALENT_NOT_VISIBLE
POD_NOT_OWNED
POD_MEMBER_NOT_FOUND
POD_LEAD_NOT_MEMBER
INTERVIEW_REQUEST_NOT_OWNED
PILOT_NOT_OWNED
INVALID_STATUS_TRANSITION
```

Public and Client responses must not reveal whether another Company's records
exist.

## 7. Transactional Rules

- Lead conversion creates Company, User, ClientAccount, and invitation context
  atomically before sending email.
- Pilot Request participant expansion and deduplication occur in one transaction.
- Pilot approval and Pilot creation occur in one transaction.
- Pilot start updates Pilot dates and Company status atomically.
- Scheduled Pilot completion updates Pilot and Company status atomically.
- Company archive/deactivation revokes or blocks access immediately.
- Status mutations record actor, timestamp, previous state, and next state.

## 8. No General Notifications

V1 sends essential email only for:

- Client account invitation
- Talent account invitation
- Talent profile completion request
- Password reset

Interview, Pilot, Pod, and progress-update mutations do not send automated
notifications in V1.
