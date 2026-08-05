# BlihOps V1 Pilot Feature

## 1. Purpose

A Pilot is a two-week trial engagement that lets a Client evaluate working with
BlihOps and selected Talent before deciding whether to enter a longer-term
engagement.

The platform supports Talent discovery, Interview Request tracking, Pilot setup,
and progress visibility. Daily work and communication are conducted manually
outside the platform.

## 2. Related Concepts

| Concept | Meaning |
|---|---|
| Pilot Request | A public expression of interest that creates a Lead for Admin review |
| Talent Pod | A private Client shortlist used to organize and compare visible Talent |
| Interview Request | A Client request for BlihOps to coordinate an interview with selected Talent |
| Pilot | The agreed two-week trial created and managed by Admin after interviews |
| Pilot Participant | Talent explicitly added by Admin to the agreed Pilot |
| Longer-term engagement | A separate post-Pilot commercial decision outside V1 |

A Pilot Request does not create a Pilot. Pod membership does not create an
Interview Request or Pilot participation. Completing an interview does not create
a Pilot automatically.

## 3. Actors

### Client

- Discovers visible, available Talent.
- Organizes possible candidates in planning-only Pods.
- Requests interviews with one or more Talent Profiles.
- Communicates with BlihOps outside the platform when deciding whether to proceed.
- Monitors an agreed Pilot through the Client Workspace.

### Admin

- Reviews Interview Requests.
- Confirms Talent availability and contacts Talent manually.
- Schedules interviews and updates their statuses.
- Creates a Pilot after the Client and BlihOps agree to proceed.
- Defines Pilot goals, dates, milestones, and participating Talent.
- Starts and manages the Pilot.
- Records client-visible progress and the final outcome.

### Talent

- Is contacted and coordinated by BlihOps outside the platform.
- Does not receive Pilot-management access in V1.
- Is not notified or assigned by Pod membership.

## 4. End-to-End Flow

```text
Client enters Workspace
→ Searches and reviews visible Talent
→ Optionally organizes Talent into Pods
→ Requests one or more interviews
→ Admin checks availability and contacts Talent manually
→ Admin schedules the interview
→ Interview Request moves from Pending to Scheduled
→ Interview takes place
→ Interview Request moves to Completed
→ [More interviews needed] → Client submits additional Interview Requests
→ [Client and BlihOps agree to proceed] → Admin creates two-week Pilot
→ Admin defines goals, dates, milestones, and participating Talent
→ Admin starts Pilot
→ Admin records progress and client-visible updates
→ Client monitors Pilot in the Workspace
→ Admin completes or cancels Pilot and records the outcome
→ Client and BlihOps decide manually whether to enter a longer-term engagement
```

## 5. Talent Pods

Pods help Clients build and compare possible Talent combinations before deciding
whom to interview.

Pods may be used for role-based shortlists, alternative candidate combinations,
comparing possible Pilot participants, and selecting a planning-only Pod Lead.

Pods do not:

- Confirm Talent availability.
- Contact or notify Talent.
- Create Interview Requests.
- Add Talent to a Pilot.
- Reserve, assign, hire, or contract Talent.

Only an explicit Interview Request begins interview coordination. Only an Admin
action adds agreed Talent to a Pilot.

## 6. Interview Requests

The Client submits an Interview Request from an available Talent Profile and may
include context for BlihOps.

| Status | Meaning |
|---|---|
| Pending | Submitted and awaiting BlihOps review |
| Scheduled | Availability is confirmed and interview scheduling is recorded |
| Completed | The interview took place |
| Cancelled | The interview will not take place |

Rules:

- BlihOps owns scheduling and all direct Talent communication.
- The Client can view but cannot directly schedule or change request status.
- The Client may request interviews with multiple Talent Profiles.
- The Client may request additional interviews after earlier interviews finish.
- A Completed request does not indicate Pilot approval or create Pilot
  participation.

## 7. Pilot Creation

Admin creates the Pilot only when:

- The Company and Client Workspace exist.
- Relevant interviews have been completed.
- The Client and BlihOps have manually agreed to proceed.
- The intended participating Talent and their availability are confirmed.

Required Pilot information:

- Company
- Name or reference
- Client-visible goals
- Client-visible scope summary
- Planned start date
- Planned end date
- Participating Talent

The planned dates define a two-week Pilot. Admin may also prepare milestones before
starting it.

## 8. Pilot Lifecycle

| Status | Meaning |
|---|---|
| Planned | The agreed Pilot is being prepared and has not started |
| Active | The two-week Pilot is underway |
| Completed | The Pilot ended and its outcome was recorded |
| Cancelled | The Pilot will not continue |

```text
Planned → Active → Completed
Planned → Cancelled
Active → Cancelled
```

Rules:

- A Company may have multiple Pilots over time.
- A Company may have at most one Planned or Active Pilot at once.
- Starting a Pilot requires goals, dates, and at least one participant.
- Completed and Cancelled Pilots remain available as history.
- A Pilot is never created or started automatically from a Lead, Pod, or Interview
  Request.

## 9. Milestones

Milestones provide lightweight progress visibility and may include a title,
client-visible description, target date, completion date, status, and display
order.

Milestone statuses:

- Pending
- In Progress
- Completed
- Cancelled

Milestones are not tasks. They do not support assignees, subtasks, comments,
dependencies, time tracking, or attachments in V1.

## 10. Participating Talent

Admin explicitly adds the Talent agreed for the Pilot.

Rules:

- A participant must reference an existing Talent Profile.
- The same Talent cannot appear twice in one Pilot.
- Pod membership does not create or remove participation.
- A hidden or archived Talent Profile remains linked to Pilot history but is shown
  as unavailable and cannot be opened by the Client.
- Participation does not create a contract, payment, or employment record.

## 11. Progress and Updates

Admin maintains Pilot progress by updating:

- Pilot status and dates
- Milestone statuses
- Participating Talent
- Client-visible updates
- Final outcome

A client-visible update may contain a title, message, effective date, and optional
related milestone. Internal notes remain Admin-only and must not appear in the
Client Workspace.

## 12. Client Workspace Experience

The Client can view:

- Current Pilot status
- Goals and scope
- Planned dates and timeline
- Milestones and progress
- Participating Talent they are permitted to access
- Client-visible updates
- Completed or Cancelled Pilot history

The Client cannot create or start a Pilot, change its status or dates, edit
milestones, manage participants, post updates, or contact Talent directly through
the platform.

When no Pilot exists, the Pilot Status screen explains that BlihOps creates a Pilot
after interviews and agreement rather than presenting an empty management tool.

## 13. Admin Experience

Pilot and Interview Request management remain within Company Detail. Admin can:

- Review Interview Requests and scheduling information.
- Create and edit the Company's Pilot.
- Start, complete, or cancel the Pilot.
- Manage goals, dates, milestones, participants, and updates.
- Record internal notes and the final outcome.
- Review Pilot activity and history.

There is no separate V1 project-management application.

## 14. Activity

Activity records capture Interview Request status changes and all important Pilot
lifecycle, milestone, participant, update, and outcome changes.

The Client's recent-activity feed may show Interview Request status changes, Pilot
creation and status changes, completed milestones, client-visible Pilot updates,
participant changes, and related Talent becoming unavailable. It excludes internal
notes, Admin-only changes, and technical events. Pilot events do not send automated
emails or create notifications in V1.

## 15. Edge States

- If requested Talent is unavailable, Admin keeps or cancels the Interview Request
  and the Client may request another Talent.
- If the Client needs more evaluation, additional Interview Requests can be made
  before Pilot creation.
- If a participant becomes unavailable before the Pilot starts, Admin updates the
  planned participants before activation.
- If the Pilot is cancelled, its history and recorded progress remain available.
- If the Company is archived, Client Workspace access is blocked while Pilot
  history is preserved.

## 16. Out of Scope

- Automatic Talent matching or availability confirmation
- Direct Client-to-Talent messaging
- Video interview hosting
- Daily task or project management
- Time tracking
- Contracts and electronic signatures
- Payments and invoicing
- Automated transition from Pilot to a longer-term engagement
