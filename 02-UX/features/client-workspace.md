# BlihOps V1 Client Workspace Experience

## 1. Purpose

The Client Workspace is the protected Web experience for one Client Account
and its Company. It is a product concept and route context, not a separate
Workspace database model.

The client enters `/client-workspace`. The API resolves the Company ID from the
authenticated Client Account and scopes every request to that Company.

## 2. Navigation

```text
Client Workspace
├── Work
├── Talents
│   └── Talent Detail
├── Pods
│   └── Pod Detail
└── Interview Requests
```

Account actions such as password reset and logout remain available globally.
There is no Client User management screen in V1.

## 3. Entry and Access States

### Available

The Client Account is `ACTIVE` and the Company is not `ARCHIVED`. The Client is
redirected to `/client-workspace` after login or invitation activation.

### Invitation Pending

The Client has not activated the account. The invitation flow is available, but
protected Workspace pages are not.

### Deactivated Client Account

The Client cannot log in or use existing sessions. The UI explains that access
has been disabled by Admin.

### Archived Company

The Client Account may remain active, but the Company-scoped Workspace is
unavailable. The UI explains that the Company Workspace has been archived.

## 4. Work Page

Work is the first page and is designed to support both Pilot preparation and the
future Active partnership stage.

### 4.1 Pilot Preparation State

When the Company is in `PILOT_PREPARATION` and has no active Pilot, the page
shows:

- Company context
- Empty Pilot state
- Explanation that the Client can prepare and submit the one free 14-day Pilot
- `Submit Pilot Request` action
- Talent and Pod summaries
- Interview Request summary

The Client may submit only one non-final Pilot Request at a time. A rejected
request can be edited and resubmitted.

### 4.2 Pilot Request In Review

The page shows:

- Submitted Pilot title
- Request status: Submitted or In Review
- Request information
- Selected participant talents
- Submitted date
- Admin review state

The Client cannot create a duplicate request while the current request is being
reviewed.

### 4.3 Planned Pilot

After Admin approval, the request creates a planned Pilot. The page shows:

- Pilot title
- Planned status
- Goals and success measurements
- Planned start date
- Expected 14-day duration
- Participants

The Client cannot start or edit the Pilot.

### 4.4 Active Pilot

When Admin starts the Pilot, the Company becomes `ON_PILOT`. The page shows:

- Pilot title
- Active status
- Start and end dates
- Days remaining
- Business goals
- Success measurements
- Participants
- Progress update timeline
- `Add Progress Update` action

The Client can only submit a progress update. The Client cannot edit official
Pilot information, participants, dates, status, or goals.

### 4.5 Completed Pilot

After the fixed 14-day period, the Pilot becomes `COMPLETED` and the Company
becomes `PILOT_COMPLETED`.

The Work page shows:

- Completed Pilot card
- Completion dates
- Pilot history
- Message encouraging the Client to start a future relationship with Blihops

Opening the card shows a read-only Pilot detail page. The Client cannot submit
new updates.

### 4.6 Active Partnership

After Admin changes the Company to `ACTIVE`, the Work page shows simple
partnership information:

- Active partnership status
- Partnership start date
- Completed Pilot count, which is at most one in V1
- Pilot history

There are no contracts, billing, tasks, time tracking, chat, or daily work
management features in V1.

### 4.7 Archived State

An archived Company cannot enter the Workspace. History is retained in the
Admin system and becomes available again only if Admin restores the Company and
the Client Account is active.

## 5. Talent Directory

The Talent Directory lists active, visible, client-eligible Talent Profiles.
For the first implementation, the directory may use approved dummy Talent data.

Client capabilities:

- Search relevant profile text
- Filter by role, skills, and experience
- Sort and paginate
- Open Talent Detail
- Add a Talent to a private Pod
- Submit an Interview Request

Talent cards may show:

- Photo
- Name
- Professional headline
- Role and experience
- Tech stack and skills
- English level
- Blihops monthly selling rate in EUR

Internal compensation, hourly rates, margin, private assessment information,
and Admin-only notes are never visible.

## 6. Talent Detail

Talent Detail may be a route-level page or a dialog in the first UI iteration.
It shows only client-visible fields and provides:

- Add to Pod
- Request Interview

The Client cannot contact or invite Talent directly. Request Interview creates
an operational request for Blihops.

## 7. Private Pods

Pods are private Client planning groups.

Client capabilities:

- Create a Pod with title and optional description
- Add visible Talent
- Remove Talent
- Assign one current member as Pod Leader
- Change or clear Pod Leader
- Rename or update description
- Request interviews for all members
- Select a Pod as a Pilot Request participant source
- Archive a Pod

Pod rules:

- A Pod may contain a Talent more than once only conceptually; duplicate
  membership is prevented.
- Pod membership does not reserve, assign, notify, contact, or add Talent to a
  Pilot.
- Removing the Pod Leader clears the leader assignment.
- If a Talent becomes unavailable, the Pod keeps an unavailable placeholder and
  blocks profile access.
- Admin cannot view or manage Pods.

When a Pod is used for an interview or Pilot Request, the system expands its
current members into individual operational records. Later Pod changes do not
modify existing requests or Pilots.

## 8. Interview Requests

A Client may request an interview with one Talent or with every current member
of a Pod.

A Pod action creates separate individual Interview Requests, grouped internally
as one client action. Admin sees the individual Talent requests and scheduling
state, but not private Pod details.

Client-visible statuses:

```text
PENDING
SCHEDULED
COMPLETED
REJECTED
CANCELLED
```

The Client can view request history, status, scheduled date/time, and any
client-visible coordination message. Blihops owns scheduling and direct Talent
communication.

Interview completion is recommended but not required for Pilot participation.
The Client may include any visible Talent in a Pilot Request.

## 9. Pilot Request Form

The Client can submit a Pilot Request only while the Company is in
`PILOT_PREPARATION` and has not completed a successful Pilot.

Fields:

- Pilot title — required
- Business goals — required
- Success measurements — required
- Expected start date — required
- Expected duration — fixed at 14 days
- Additional context — optional
- Individual Talent participants — optional
- Pod participant sources — optional

Selecting a Pod expands its current members into individual participants. A
Talent selected both individually and through a Pod appears once.

Interview completion is not a validation requirement.

## 10. Progress Updates

During an active Pilot, the Client can submit a minimal permanent record:

- Update text
- Author — automatic
- Created date/time — automatic

The Client cannot edit or delete an update. Admin may remove an update after
confirmation. Removal should remain represented in internal audit history.

## 11. Client Visibility Rules

The Client can see:

- Their Company information
- Visible Talent Profiles
- Their private Pods
- Their Interview Requests
- Their Pilot Request status
- Their Pilot information
- Their own client-visible Pilot history and updates

The Client cannot see:

- Internal Admin notes
- Talent compensation or margin
- Other Companies
- Private Talent assessment information
- Internal audit history
- Other Clients' Pods or requests

## 12. Out of Scope

- Client team members
- Direct Talent messaging
- Contracts and payments
- Daily tasks or project management
- Milestones
- Time tracking
- Client-managed Pilot status or participants
- General notifications or notification center
