# BlihOps V1 Client and Company Feature — Implementation Plan

## 1. Purpose

This is the implementation sequence for the Client and Company feature. It is
ordered by dependency:

```text
BlihOps API → BlihOps Admin → BlihOps Web Client Workspace → Integration and QA
```

The API must establish the data model, authorization, state transitions, and
endpoint contracts before either frontend implements the feature.

Related design documents:

- `01-Client-Company-Model.md`
- `02-Data-Model.md`
- `03-API-Design.md`
- `04-Architecture.md`
- `../../02-UX/features/client-company.md`
- `../../02-UX/features/client-workspace.md`
- `../../02-UX/features/pilot.md`

## 2. Non-Negotiable V1 Decisions

- There is no `ClientWorkspace` database model.
- `Company` is the client data and authorization scope.
- `ClientAccount` connects one Client User to one Company.
- The Client route is `/client-workspace`; it does not accept a Workspace ID.
- Client Account statuses are `PENDING_INVITATION`, `ACTIVE`, and `DEACTIVATED`.
- Company statuses are `PILOT_PREPARATION`, `ON_PILOT`,
  `PILOT_COMPLETED`, `ACTIVE`, and `ARCHIVED`.
- Only `PILOT_PREPARATION` allows a Client Pilot Request.
- Interviews are recommended but not required for Pilot participation.
- Pods are completely private to the Client.
- A Pod action creates individual operational Interview Requests or Pilot
  participants; Admin never sees private Pod content.
- Each Company has one free 14-day Pilot opportunity.
- Rejected requests can be edited and resubmitted.
- Cancelled Pilots do not consume the free Pilot opportunity.
- Only a completed Pilot consumes the opportunity.
- There are no milestones in V1.
- Client progress updates contain text, automatic author, and automatic timestamp.
- Progress updates are immutable; only Admin can remove them.
- Active companies show partnership state and Pilot history only.
- Contracts, billing, tasks, chat, and daily work management remain out of scope.

## 3. Implementation Strategy

Implement vertical slices in this order:

1. API persistence and authorization foundation.
2. API Company onboarding and invitation flow.
3. API Client and Admin read/write operations.
4. Admin Company operations and Pilot oversight.
5. Client Workspace entry and Work page.
6. Client Talent, Pod, Interview, and Pilot Request flows.
7. Active Pilot updates and completed/active history.
8. Cross-application verification and deployment readiness.

Do not build frontend screens against guessed response shapes. Add the API
contract and tests first, then consume the contract from Admin and Web.

---

# Phase 0 — Preparation and Contracts

## 0.1 Confirm repository boundaries

- [ ] API changes stay in `blihops-api`.
- [ ] Admin changes stay in `blihops-admin`.
- [ ] Client Workspace changes stay in `blihops-web`.
- [ ] Design documents remain in `blihops-design`.
- [ ] Do not create a Workspace Prisma model.
- [ ] Preserve the existing Better Auth User/Session/Account ownership.

## 0.2 Confirm existing integration points

- [ ] Reuse the existing Better Auth invitation acceptance route.
- [ ] Extend the existing invitation service so a Client invitation creates and
      tracks `ClientAccount` state.
- [ ] Reuse existing email factory and invitation template infrastructure.
- [ ] Follow existing Leads controller/repository/service/router/schema patterns.
- [ ] Follow existing Admin TanStack Query and section-component patterns.
- [ ] Follow existing Web locale and protected-route patterns.

## 0.3 Establish implementation vocabulary

Use these terms consistently in code and UI:

- `Company` — business record and scope.
- `ClientAccount` — User-to-Company access relationship.
- `Client Workspace` — Web product experience, not a database model.
- `PilotRequest` — Client proposal awaiting Admin decision.
- `Pilot` — approved trial record.
- `ClientPod` — private Client planning group.
- `InterviewRequest` — Blihops coordination request for one Talent.
- `PilotProgressUpdate` — immutable Client note during an active Pilot.

---

# Phase 1 — Blihops API

The API is implemented first because it owns persistence, authorization, status
transitions, invitation activation, Company scope, and the 14-day completion job.

## 1.1 Add Prisma enums and models

File:

```text
blihops-api/prisma/schema.prisma
```

Add the models and enums defined in `02-Data-Model.md`:

- `ClientAccount`
- `ClientInvitation`
- `Company`
- `PilotRequest`
- `PilotRequestParticipant`
- `Pilot`
- `PilotParticipant`
- `PilotProgressUpdate`
- `ClientPod`
- `ClientPodMember`
- `InterviewRequestGroup`
- `InterviewRequest`
- Client Account, Company, invitation, request, Pilot, interview, Pod, and
  participant enums.

Add the required relations to the existing `User` and `Lead` models.

Implementation constraints:

- `ClientAccount.userId` is unique.
- `ClientAccount.companyId` is unique for V1.
- `PilotRequestParticipant` prevents duplicate Talent per request.
- `PilotParticipant` prevents duplicate Talent per Pilot.
- Pod membership uses a composite primary key.
- Do not add `workspaceId`.
- Do not store raw invitation tokens.

Run:

```bash
cd blihops-api
pnpm db:format  # if available in the local Prisma setup
pnpm db:generate
pnpm db:migrate
```

If the repository does not expose `db:format`, use the project-approved Prisma
format command before generating and migrating.

## 1.2 Create API feature modules

Recommended feature structure:

```text
blihops-api/src/features/client-company/
├── index.ts
├── company.controller.ts
├── company.repository.ts
├── company.router.ts
├── company.schema.ts
├── company.service.ts
├── client-account.controller.ts
├── client-account.repository.ts
├── client-account.router.ts
├── client-account.schema.ts
├── client-account.service.ts
├── interview.controller.ts
├── interview.repository.ts
├── interview.router.ts
├── interview.schema.ts
├── interview.service.ts
├── pod.controller.ts
├── pod.repository.ts
├── pod.router.ts
├── pod.schema.ts
├── pod.service.ts
├── pilot.controller.ts
├── pilot.repository.ts
├── pilot.router.ts
├── pilot.schema.ts
├── pilot.service.ts
└── pilot-completion.job.ts
```

The exact split may follow the existing repository convention, but each module
must keep validation, persistence, business rules, and transport concerns
separate.

## 1.3 Implement Company conversion

Implement the Admin conversion transaction:

```text
qualified Lead
→ duplicate Company check
→ Company(PILOT_PREPARATION)
→ User(role=CLIENT)
→ ClientAccount(PENDING_INVITATION)
→ ClientInvitation(PENDING)
→ Lead(CONVERTED)
→ commit
→ issue/send invitation email
```

Requirements:

- Conversion must be idempotent or reject a duplicate attempt safely.
- Company, User, ClientAccount, invitation context, and Lead conversion must not
  partially commit.
- Email failure must not create duplicate records.
- Admin can retry the invitation after a delivery failure.
- Store the Lead-to-Company relationship for traceability.

## 1.4 Extend invitation and activation

Update the existing Auth invitation flow:

- Create Client Account state when inviting a Client.
- Store invitation lifecycle state separately from Client Account state.
- Mark invitation `ACCEPTED` and Client Account `ACTIVE` after successful
  password creation.
- Reject expired, replaced, consumed, or invalid invitations.
- Resend replaces the previous pending invitation rather than creating another
  Client User or Client Account.
- Deactivation invalidates current sessions and blocks future login.
- Reactivation is an explicit Admin operation.

Tests must cover:

- First invitation.
- Resend pending invitation.
- Replace expired invitation.
- Accept valid invitation.
- Reject reused invitation.
- Reject expired invitation.
- Deactivated Client login/session behavior.

## 1.5 Implement Company authorization context

Create a server-side Client context helper/middleware that:

```text
session.user.id
→ ClientAccount
→ companyId
→ Company
```

It must reject:

- Non-Client roles on Client endpoints.
- Missing ClientAccount.
- `PENDING_INVITATION` accounts.
- `DEACTIVATED` accounts.
- Archived Companies.
- Any requested Company or resource not owned by the resolved Company.

Every Client repository query must include the resolved `companyId`. Never trust
Company IDs supplied by the Client as an authorization source.

## 1.6 Implement Admin Company endpoints

Implement the endpoints in `03-API-Design.md` for:

- Company list, filtering, pagination, and search.
- Company creation and Lead conversion.
- Company detail projection.
- Company edits.
- Invitation resend/replace.
- Client Account deactivate/reactivate.
- Company archive/restore.
- Company activity and notes.

The detail projection must include separate values for:

- Company status.
- Client Account status.
- Invitation status.
- Derived Workspace access state.
- Pilot Request/Pilot summaries.
- Interview Request summaries.

It must not include private Pod records.

## 1.7 Implement Client Workspace context endpoints

Implement:

```text
GET /api/v1/client/me
GET /api/v1/client/work
```

`/client/work` must return a status-aware projection for:

- Pilot Preparation.
- Submitted/In Review Pilot Request.
- Rejected/resubmittable request.
- Planned Pilot.
- Active Pilot.
- Completed Pilot history.
- Active partnership.

The projection must be generated from the authenticated Client Account and
Company ID. No Workspace ID is required.

## 1.8 Implement Talent directory endpoints

Implement Client-safe Talent endpoints:

```text
GET /api/v1/client/talents
GET /api/v1/client/talents/:talentProfileId
```

Requirements:

- Return only active, visible, client-eligible Talent.
- Support search, filters, sorting, and pagination.
- Exclude compensation, internal notes, assessments, margin, and Admin-only
  fields.
- Reject hidden, archived, or unavailable profile access.
- Initially support dummy/seeded Talent data if the Talent Profile feature is not
  yet complete, but keep the response contract stable.

## 1.9 Implement private Pod endpoints

Implement Client-only CRUD and membership operations:

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

Requirements:

- Scope every operation to the Client Company.
- Prevent duplicate membership.
- Require Pod Lead to be a current member.
- Do not expose Pods from any Admin endpoint.
- Preserve unavailable Talent as a non-clickable placeholder where needed.
- Expand Pod membership into individual request/participant snapshots for
  interview and Pilot operations.

## 1.10 Implement Interview Request endpoints

Implement:

```text
GET  /api/v1/client/interview-requests
POST /api/v1/client/interview-requests
GET  /api/v1/client/interview-requests/:requestId
GET  /api/v1/admin/companies/:companyId/interview-requests
GET  /api/v1/admin/interview-requests/:requestId
PATCH /api/v1/admin/interview-requests/:requestId
```

Requirements:

- Individual request from Talent Detail.
- Grouped request creation from a Pod, producing one request per Talent.
- Independent status and scheduling for each Talent request.
- Admin controls status, date, time, timezone, client-visible message, and
  internal notes.
- Client cannot directly schedule or mutate status.
- Admin sees individual Talent records, not private Pod details.

## 1.11 Implement Pilot Request and Pilot endpoints

Client endpoints:

```text
GET   /api/v1/client/pilot-requests
POST  /api/v1/client/pilot-requests
GET   /api/v1/client/pilot-requests/:requestId
PATCH /api/v1/client/pilot-requests/:requestId
POST  /api/v1/client/pilot-requests/:requestId/resubmit
GET   /api/v1/client/pilots
GET   /api/v1/client/pilots/:pilotId
POST  /api/v1/client/pilots/:pilotId/updates
GET   /api/v1/client/pilots/:pilotId/updates
```

Admin endpoints:

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
```

Required rules:

- Only `PILOT_PREPARATION` allows Client submission.
- Only one non-final Pilot Request/Pilot process may exist at a time.
- Rejected requests can be edited and resubmitted.
- Approval creates `PLANNED`; it does not start the Pilot.
- Starting sets actual dates and moves Company to `ON_PILOT`.
- Duration is always 14 days and is server-controlled.
- Cancellation returns the Company to `PILOT_PREPARATION`.
- Only `COMPLETED` consumes the free Pilot opportunity.
- Client update creation is allowed only while Pilot is `ACTIVE`.
- Progress updates have no update endpoint; Admin removal is soft-delete only.
- There are no milestone endpoints.

## 1.12 Implement scheduled Pilot completion

Add an idempotent scheduled job:

```text
Find ACTIVE Pilots where endDate <= now
→ lock Pilot and Company
→ ACTIVE → COMPLETED
→ ON_PILOT → PILOT_COMPLETED
→ write audit events
```

Requirements:

- Re-running the job does not duplicate transitions or activity.
- Cancellation wins if it occurred before the job lock.
- Admin can manually complete only as an exceptional operation.
- No general completion email is sent in V1.

## 1.13 Implement activity and audit

Record:

- Company conversion, edit, archive, and restore.
- Invitation send, resend, replacement, acceptance, and expiry.
- Client Account activation, deactivation, and reactivation.
- Pilot Request submission, review, rejection, resubmission, and approval.
- Pilot creation, start, cancellation, completion, and outcome.
- Interview Request creation and status changes.
- Progress Update creation and Admin removal.

Create a separate Client-safe activity projection. Never expose the internal audit
stream directly to the Client.

## 1.14 API tests and acceptance checks

Add unit and integration coverage for:

- Schema constraints and conversion transaction.
- Invitation lifecycle.
- Company/Client access matrix.
- Archived Company and deactivated Client behavior.
- Company-scoped ownership/IDOR protection.
- Private Pod non-exposure.
- Interview request batch expansion.
- Pilot Request status transitions.
- Rejected request resubmission.
- Pilot cancellation preserving eligibility.
- Completed Pilot consuming eligibility.
- Fixed 14-day end date.
- Idempotent completion job.
- Immutable progress updates.

Run from `blihops-api`:

```bash
pnpm lint
pnpm typecheck
pnpm test:all
pnpm format:check
pnpm build
```

Do not begin Admin implementation until the API tests cover the core state and
authorization rules.

---

# Phase 2 — BlihOps Admin

Admin is implemented second because it is the operational control center and
must consume the stable API contracts.

## 2.1 Add Admin API types and query keys

Recommended areas:

```text
blihops-admin/src/lib/api/
blihops-admin/src/lib/queries/
blihops-admin/src/lib/mutations/
```

Add typed contracts and TanStack Query keys for:

- Companies list/detail.
- Client Account/invitation state.
- Interview Requests.
- Pilot Requests.
- Pilots and progress updates.
- Company activity and notes.

Invalidate related queries after mutations:

- Company conversion → Companies, Leads, Company Detail.
- Invitation action → Company Detail.
- Account/company access action → Company Detail and list row.
- Pilot transition → Company Detail, Dashboard, Pilot sections.
- Interview status update → Company Detail and related Client projection.

## 2.2 Replace the Companies placeholder page

Current location:

```text
blihops-admin/src/app/(dashboard)/companies/page.tsx
```

Implement:

- Company list.
- Search.
- Company status filter.
- Client Account status filter.
- Invitation/access status display.
- Pagination if required by API.
- Row click to Company Detail.
- Empty, loading, error, and retry states.

Do not show a Workspace ID. Show derived access state such as:

```text
Available
Pending invitation
Client deactivated
Company archived
```

## 2.3 Implement Company creation and Lead conversion

Add the Admin form/action for:

- Manual Company creation.
- Conversion from Lead.
- Company information.
- Client representative full name.
- Client representative email.
- Duplicate Company warning.
- Confirmation before creating and sending invitation.

After success:

- Lead is marked converted.
- Company Detail opens.
- Invitation status is visible.
- Failed email delivery shows a truthful retry action.

## 2.4 Implement Company Detail overview

Recommended component areas:

```text
blihops-admin/src/components/sections/companies/
├── companies-page.tsx
├── company-detail-page.tsx
├── company-overview.tsx
├── client-account-card.tsx
├── company-access-card.tsx
├── company-status-actions.tsx
├── invitation-actions.tsx
└── company-activity.tsx
```

Overview must show separate cards for:

### Company

- Name and profile fields.
- Relationship status.
- Archive/restore action.
- Pilot eligibility state.

### Client Account

- Name.
- Email.
- Account status.
- Invitation status.
- Invited/activated dates.
- Last login when available.
- Resend/replace invitation.
- Deactivate/reactivate.

### Workspace access

Derived reason:

```text
Available
Blocked — invitation pending
Blocked — Client deactivated
Blocked — Company archived
```

## 2.5 Implement Admin Interview Requests

Keep Interview Requests inside Company Detail.

Admin must be able to:

- List individual requests.
- Filter by status.
- View Talent and Company context.
- See grouped-action indication without private Pod details.
- Set status.
- Set date, time, and timezone.
- Add client-visible message.
- Add internal notes.
- Reject or cancel with a reason.

Do not add a Pod management view.

## 2.6 Implement Admin Pilot Request review

Company Detail must support:

- Pilot Request list/history.
- Submitted and In Review states.
- Full request fields.
- Participant list.
- Individual/POD-origin indication only where safe.
- Rejection reason.
- Approve action.
- Resubmission history/activity.

Approval should display the resulting `PLANNED` Pilot and not automatically mark
it Active.

## 2.7 Implement Admin Pilot management

Admin can:

- Review planned Pilot details.
- Edit title, goals, success measurements, dates, and participants.
- Start the Pilot.
- Cancel the Pilot with reason.
- View active Pilot data.
- View Client progress updates.
- Remove a progress update with confirmation.
- View completed/cancelled history.
- Mark Company Active or Archived after completion.

The UI must make the distinction explicit:

```text
Approve Request → Pilot PLANNED
Start Pilot     → Pilot ACTIVE + Company ON_PILOT
Complete        → Pilot COMPLETED + Company PILOT_COMPLETED
```

There is no milestone editor.

## 2.8 Update Admin Dashboard summaries

Add or update summary cards for:

- Companies in Pilot Preparation.
- Pending Client invitations.
- Pilot Requests in review.
- Planned/Active Pilots.
- Completed Pilots awaiting Active/Archived decision.
- Interview Requests requiring attention.

Do not add general notifications or an unread notification center.

## 2.9 Admin states and accessibility

Every Company and Pilot screen needs:

- Loading state.
- Empty state.
- Error/retry state.
- Disabled action state.
- Confirmation for archive, deactivate, cancel, and remove.
- Clear status text not dependent on color alone.
- Keyboard-accessible tables, forms, dialogs, and actions.

## 2.10 Admin verification

Run from `blihops-admin`:

```bash
pnpm lint
pnpm typecheck
pnpm format:check
pnpm build
```

Manually verify:

- Lead conversion opens Company Detail.
- Invitation status updates correctly.
- Deactivated Client and archived Company show the correct access reason.
- Private Pods never appear.
- Pilot Request approval creates Planned, not Active.
- Pilot start updates Company status.
- Completed Pilot enables Active/Archived decisions.

---

# Phase 3 — BlihOps Web Client Workspace

The Web client is implemented after the API and Admin workflows are stable.

## 3.1 Replace the Workspace route shape

Current scaffold:

```text
blihops-web/src/app/[locale]/client-workspace/[workspaceId]/
```

Implement the Company-scoped route without a Workspace ID:

```text
blihops-web/src/app/[locale]/client-workspace/
├── layout.tsx
├── page.tsx
├── talents/
│   └── [talentProfileId]/page.tsx
├── pods/
│   ├── page.tsx
│   └── [podId]/page.tsx
└── interview-requests/page.tsx
```

The server/API resolves Company scope from the authenticated Client Account.
Remove any client authorization that depends on a URL `workspaceId`.

## 3.2 Update role redirect and access guards

After Client login or invitation activation:

```text
CLIENT → /client-workspace
```

Implement states for:

- Active Client + available Company.
- Pending invitation.
- Deactivated Client Account.
- Archived Company.
- Missing ClientAccount.
- Unauthorized role.

The API remains the source of truth; frontend guards improve UX but do not
replace server authorization.

## 3.3 Implement the Work page

Use the `/api/v1/client/work` projection to render:

### Pilot Preparation

- Empty explanation.
- Submit Pilot Request action.
- Talent, Pod, and Interview Request summaries.

### Request Submitted/In Review

- Request information.
- Status.
- Submission date.
- Participant list.
- No duplicate submit action.

### Rejected

- Rejection reason.
- Editable request form.
- Resubmit action.

### Planned

- Official planned information.
- Start date.
- 14-day duration.
- Read-only participant list.

### Active

- Title/status.
- Start/end dates and days remaining.
- Goals and success measurements.
- Participants.
- Progress update timeline.
- Add Progress Update form.

### Completed

- Completed Pilot card.
- Read-only detail.
- Pilot history.
- Future partnership call-to-action.
- No progress update or Pilot Request action.

### Active partnership

- Simple partnership statistics.
- Pilot history.
- No contracts/tasks/billing UI.

## 3.4 Implement Talent Directory and Detail

Use the Client Talent API contract.

Implement:

- Search.
- Filters.
- Sort.
- Pagination.
- Talent cards.
- Talent Detail page or dialog.
- Add to Pod action.
- Request Interview action.
- Client-safe fields only.

Dummy Talent data is acceptable for the first UI slice, but the component should
use the final API shape.

## 3.5 Implement private Pods

Implement:

- Pod list.
- Create Pod.
- Edit title/description.
- Add/remove Talent.
- Assign/change/clear Pod Lead.
- Archive Pod.
- Request interviews for all Pod members.
- Select Pod in Pilot Request.

Client UX must never imply that Pod membership reserves or contacts Talent.
Admin does not have a Pod UI.

## 3.6 Implement Interview Requests

Implement:

- Individual request from Talent Detail.
- Group request from Pod.
- Request context.
- Request list.
- Individual statuses.
- Scheduled date/time display.
- Client-safe coordination message.
- Empty/loading/error states.

The client cannot edit status or schedule.

## 3.7 Implement Pilot Request form

Form fields:

- Pilot title.
- Business goals.
- Success measurements.
- Expected start date.
- Fixed duration display: 14 days.
- Additional context.
- Individual Talent selection.
- Pod selection.

On submit:

- Prevent duplicate submission.
- Display the resulting request status.
- Display expanded participant list.
- Do not require completed interviews.
- Do not allow duration editing.

## 3.8 Implement Active Pilot progress updates

The Client can submit only:

```text
Update text
```

Author and date are returned by the API. The UI must not show edit/delete actions
for Client-created updates.

After completion, remove the Add Update action and show read-only history.

## 3.9 Web verification

Run from `blihops-web`:

```bash
pnpm lint
pnpm typecheck
pnpm format:check
pnpm build
```

Manually verify:

- Client redirects to `/client-workspace` without a Workspace ID.
- Another Company cannot be accessed by changing a URL or request body.
- Private Pods are usable but never visible to Admin.
- Pilot Request is available only in Pilot Preparation.
- Rejected request can be edited/resubmitted.
- Active Pilot accepts updates only.
- Completed Pilot is read-only.
- Active partnership shows history and simple stats.
- Archived Company shows unavailable state.

---

# Phase 4 — Cross-Application Integration

## 4.1 End-to-end onboarding

Test the complete flow:

```text
Lead
→ Admin conversion
→ Company + Client User + ClientAccount
→ Invitation email
→ Client acceptance
→ Client /client-workspace
```

Verify the Lead, Company, ClientAccount, invitation, session, and redirect states
are consistent.

## 4.2 End-to-end optional interview flow

Test:

```text
Client Talent Detail
→ Individual Interview Request
→ Admin schedules/rejects/completes
→ Client sees safe status
```

And:

```text
Client Pod
→ Request interviews for Pod
→ Individual grouped requests
→ Admin sees talents, not Pod details
```

## 4.3 End-to-end Pilot flow

Test:

```text
Client submits Pilot Request
→ Admin In Review
→ Admin rejects
→ Client edits/resubmits
→ Admin approves
→ Pilot PLANNED
→ Admin starts
→ Company ON_PILOT
→ Client submits permanent update
→ 14-day job completes Pilot
→ Company PILOT_COMPLETED
→ Admin marks ACTIVE or ARCHIVED
```

Also test:

```text
Planned/Active Pilot
→ Admin cancels
→ Company PILOT_PREPARATION
→ Free Pilot opportunity remains available
```

## 4.4 Access matrix verification

| Scenario | Expected result |
|---|---|
| Pending invitation | Cannot enter Client Workspace |
| Active Client + Preparation Company | Workspace available; Pilot Request available |
| Active Client + On Pilot Company | Workspace available; Pilot Request unavailable |
| Active Client + Pilot Completed Company | Read-only history; Pilot Request unavailable |
| Active Client + Active Company | Partnership stats/history; Pilot Request unavailable |
| Active Client + Archived Company | Workspace blocked |
| Deactivated Client + any Company | Login/session/API blocked |
| Client changes Company ID in request | API rejects/ignores unauthorized scope |
| Admin requests Pod data | No Pod data returned |

## 4.5 Data and activity verification

- Every important transition has an audit event.
- Client activity contains only the approved client-safe event set.
- Progress update author/date cannot be changed.
- Admin removal does not expose the removed update to the Client.
- Invitation resend does not create duplicate users/accounts.
- Pilot completion is idempotent.

---

# Phase 5 — Deployment and Operational Readiness

## 5.1 Database

- [ ] Review migration against staging data.
- [ ] Confirm foreign-key behavior and indexes.
- [ ] Run `pnpm db:deploy` in deployment environment.
- [ ] Run `pnpm db:generate` during API build/deploy.
- [ ] Verify scheduled Pilot completion job is registered and observable.

## 5.2 Environment and email

- [ ] Confirm invitation redirect URL uses the correct Web origin.
- [ ] Confirm email template variables for Client invitations.
- [ ] Confirm resend/replacement behavior.
- [ ] Confirm email delivery failure logging and Admin retry behavior.

## 5.3 Monitoring

Log and monitor:

- Company conversion failures.
- Invitation send failures.
- Client activation failures.
- Authorization denials.
- Pilot transition failures.
- Scheduled completion job failures.
- Duplicate/idempotency conflicts.

Never log raw invitation tokens, passwords, or internal commercial data.

## 5.4 Release checks

API:

```bash
cd blihops-api
pnpm check
```

Admin:

```bash
cd blihops-admin
pnpm check
```

Web:

```bash
cd blihops-web
pnpm check
```

Then complete the manual end-to-end matrix from Phase 4.

---

# 6. Suggested Commit Sequence

Keep implementation commits small and dependency-ordered:

```text
feat(api): add client company and pilot schema
feat(api): add client account invitation lifecycle
feat(api): add company conversion and access authorization
feat(api): add interview and private pod endpoints
feat(api): add pilot requests and pilot lifecycle
feat(api): add pilot completion job and audit events
feat(admin): add companies list and conversion flow
feat(admin): add company detail and client access controls
feat(admin): add interview and pilot management
feat(web): replace workspace id route with company-scoped workspace
feat(web): add client Work page and pilot request flow
feat(web): add talent, pod, and interview flows
feat(web): add active pilot updates and history
 test: verify client company end-to-end workflows
```

Use the repository's conventional commit rules and keep each commit buildable
where practical.

# 7. Definition of Done

The feature is ready for V1 when:

- Lead conversion creates Company, Client User, ClientAccount, and invitation
  without creating a Workspace record.
- Client invitation activation and deactivation work correctly.
- Company archive blocks the Client Workspace.
- Client data is always scoped by session-derived Company ID.
- Client can browse visible Talent and privately manage Pods.
- Optional individual and Pod-based Interview Requests work.
- Client can submit, revise after rejection, and track one Pilot Request.
- Admin can approve, start, cancel, and manage the Pilot.
- Pilot automatically completes after 14 days.
- Only a completed Pilot consumes the free opportunity.
- Client can submit immutable progress updates only during an active Pilot.
- Completed and Active states display the correct read-only/history experience.
- Admin never sees private Pod content.
- API, Admin, and Web checks pass.
- End-to-end access, status, privacy, and failure cases are covered by tests or
  documented manual verification.
