# BlihOps V1 Talent Application and Profile — Architecture

## 1. Purpose

This document defines the runtime flows and boundaries for Talent Applications,
Profile Completion Requests, Talent Profile creation, invitations, Talent Portal
access, and Client-safe Profile delivery.

## 2. Component Boundaries

```text
blihops-web
├── Public Talent Application
├── Unauthenticated Profile Completion
└── Authenticated Talent Portal
        │
        ▼
 blihops-api
 ├── Talent Applications
 ├── Completion Requests
 ├── Talent Profiles
 ├── Talent Accounts
 ├── Invitation integration
 ├── Client-safe Talent reads
 └── Activity/audit
        │
        ▼
 PostgreSQL + protected file storage

blihops-admin ───────────────► blihops-api Admin endpoints
```

## 3. Public Application Flow

```text
Candidate opens /talent/apply
→ Web validates form and resume
→ API validates again
→ API stores immutable TalentApplication(NEW)
→ API stores private resume reference
→ Web shows confirmation
```

The public payload does not contain status, account, Profile, screening, or
Admin fields.

## 4. Admin Pipeline Flow

```text
Admin opens Applications
→ Reviews immutable submission and resume
→ Adds/edits Admin-only internal notes
→ Moves current status flexibly
→ [Approved] → sends completion request
→ [Rejected] → may later revert to any workflow stage
```

There is no assessment entity. Moving the status is the Admin's pipeline
decision. There is no stage-history record or required transition reason.

## 5. Completion Request Flow

```text
Application=APPROVED
→ Admin requests completion
→ API creates 7-day single-use request
→ Email sends secure link
→ Candidate opens unauthenticated form
→ API validates token
→ Candidate submits photo + short bio
→ API updates Application completion fields
→ Application=COMPLETION_SUBMITTED
→ Token consumed
```

The completion request does not create a Profile or account. If the candidate
needs to improve the data later, they do so through the Talent Portal after
activation.

## 6. Profile Creation Transaction

```text
Admin opens COMPLETION_SUBMITTED Application
→ Reviews combined Round 1 + Round 2 data
→ Provides required Admin fields
→ Begins transaction
→ Create TalentProfile
→ Set isVerified=true
→ Set visibility=HIDDEN
→ Create User(role=TALENT)
→ Create TalentAccount(PENDING_INVITATION)
→ Create invitation lifecycle record
→ Set Application=PROFILE_CREATED
→ Commit
→ Issue/send invitation email
```

The invitation is sent after commit. If email delivery fails, Admin can resend;
Profile and Account data remain valid and no duplicate User is created.

## 7. Invitation Activation Flow

```text
Talent opens invitation link
→ /auth/accept-invitation
→ Token validated
→ Talent sets password
→ TalentAccount=ACTIVE
→ Invitation=ACCEPTED
→ Redirect to /talent-portal
```

The Talent Portal has no Profile ID. The API resolves the Profile from the
session User ID.

## 8. Talent Portal Access Flow

```text
Talent opens /talent-portal
→ API reads session User ID
→ Loads TalentAccount
→ Loads TalentProfile
→ Requires TalentAccount=ACTIVE
→ Returns own Profile only
```

If the Account is pending or deactivated, access is denied. Talent cannot request
another Profile or provide a Profile ID to widen access.

## 9. Talent Profile Update Flow

```text
Talent edits permitted fields
→ API verifies ownership and allowed fields
→ API validates required values
→ API saves update
→ Profile change is immediately reflected
→ Activity records Talent origin
```

Talent updates are allowed while Profile visibility is `VISIBLE`, provided the
update does not invalidate required Client-visible values.

## 10. Admin Visibility Flow

### Publish

```text
Admin opens hidden Profile
→ Clicks Make Visible
→ API runs Client-visible completeness validator
→ [Failure] return missing fields; remain HIDDEN
→ [Success] set VISIBLE
```

### Edit

```text
Admin opens visible Profile
→ Any edit action is rejected
→ UI explains: hide Profile before editing
→ Admin changes VISIBLE → HIDDEN
→ Admin edits and saves
→ Admin runs publish validation again
```

This rule applies to every Admin Profile field, not only client-visible fields.

### Deactivate

```text
Admin deactivates Talent Account
→ Revoke sessions
→ Account=DEACTIVATED
→ Profile visibility=HIDDEN
→ Client-safe query excludes Profile
```

Reactivation sets Account `ACTIVE` but leaves Profile `HIDDEN`.

## 11. Client-safe Profile Flow

```text
Client requests Talent Directory
→ API filters active Talent Accounts
→ API filters visible Profiles
→ API runs/relies on visibility invariant
→ API removes Admin-only fields
→ Client receives safe Profile cards
```

The Talent feature must expose stable client-safe field names so Client/Company
can consume them later without querying Application or TalentAccount data.

## 12. File Handling

- Public Application uploads a required resume to protected storage.
- Completion form uploads a required profile photo to protected storage.
- Talent Portal can replace permitted photo/resume files.
- API validates file type and size.
- New file reference is committed before old file cleanup.
- Private file access is authorized per Application, Profile, or Talent Account.

## 13. Activity Boundaries

Admin-visible internal data:

- Application notes.
- Profile internal notes.
- Talent-originated Profile update indication.
- Invitation/account actions.

Client-visible data:

- Only visible Profile fields.
- Verification indicator.
- Talent-originated changes are reflected but internal notes and application
  information are excluded.

No status-change history record is created for Application status transitions.

## 14. Integration Points for Client/Company

After Talent Profile implementation:

- Client Directory queries `/api/v1/client/talents`.
- Client Detail queries `/api/v1/client/talents/:profileId`.
- Pods store `talentProfileId`.
- Interview Requests store `talentProfileId`.
- Pilot participant snapshots store `talentProfileId`.
- Hidden/deactivated Profiles are excluded or shown as unavailable historical
  placeholders where the Client feature requires it.

## 15. Failure and Edge States

- Completion token expired: Admin replaces pending request.
- Completion token consumed: form cannot be reopened.
- Profile creation fails: transaction rolls back.
- Invitation email fails: Account remains pending; Admin retries.
- Visible Profile edit attempted by Admin: API rejects.
- Visible Profile publish checklist fails: remains hidden.
- Talent update attempts Admin field: API rejects.
- Talent deactivated: session revoked and Profile hidden.
- Talent reactivated: Account active, Profile remains hidden.
- Client requests hidden Profile: API returns unavailable/not found safely.

## 16. Out of Scope

- Candidate self-service account creation.
- Assessment result entities.
- Stage-change audit history.
- Profile Archive status.
- Verification review workflow.
- Availability, earliest start date, or preferred engagement.
- Talent Dashboard.
- Client editing or messaging.
