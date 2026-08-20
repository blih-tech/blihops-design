# BlihOps V1 Talent Application and Profile — Domain Model

## 1. Purpose

This document is the source of truth for Talent Applications, Profile Completion
Requests, Talent Profiles, Talent Accounts, invitation activation, visibility,
and Talent Portal behavior.

## 2. Core Relationships

```text
TalentApplication
 ├── ProfileCompletionRequest(s)
 └── TalentProfile (after completion submission)
        └── TalentAccount
               └── User(role=TALENT)
```

The original Application remains immutable after submission. The Profile is a
separate operational record populated from the Application and completion data.

## 3. Application Lifecycle

```text
NEW
UNDER_REVIEW
SCREENING
TECHNICAL_ASSESSMENT
ENGLISH_ASSESSMENT
REMOTE_READINESS_ASSESSMENT
APPROVED
COMPLETION_REQUESTED
COMPLETION_SUBMITTED
PROFILE_CREATED
REJECTED
ARCHIVED
```

Admin status changes are flexible:

- Admin may move between workflow stages.
- Admin may move backward.
- `REJECTED` may be reverted to any active workflow stage.
- No status-change reason is required.
- No status-change history record is created.

Hard gates:

- Completion request requires `APPROVED`.
- Completion submission is produced by the form.
- Profile creation requires `COMPLETION_SUBMITTED`.
- `PROFILE_CREATED` is set by the Profile creation action.

## 4. Application Data Rules

Round 1 contains the Admin review data:

- Full name, work email, phone, country, city.
- Primary role, tech stack, secondary skills.
- Years of experience.
- Portfolio, GitHub, LinkedIn.
- Resume.

Screening responses are not part of the model.

Original submitted fields are immutable. The Application has one Admin-only
internal notes field that may be edited separately.

There are no assessment records, score records, pass/fail records, or stage
history records. Moving the current status is the Admin's assessment decision.

## 5. Completion Request Rules

After `APPROVED`, Admin sends a Profile Completion Request.

Round 2 contains:

- Profile photo.
- Short bio.
- Professional headline.

The completion token:

- Is single-use.
- Expires after 7 days.
- Is unauthenticated.
- Can be resent/replaced while pending or expired.
- Is consumed on successful submission.

After submission, Admin creates the Profile. There is no correction loop; Talent
can update permitted fields after activation.

## 6. Profile Creation Rules

Profile creation requires:

```text
Application = COMPLETION_SUBMITTED
```

Admin must provide:

- Seniority.
- English level.
- Client monthly selling rate.
- Assessment summary.
- Internal notes.

Creation sets:

```text
TalentProfile.visibility = HIDDEN
TalentProfile.isVerified = true
TalentAccount.status = PENDING_INVITATION
Application.status = PROFILE_CREATED
```

Profile, User, TalentAccount, invitation context, and Application status update
are created in one transaction. The invitation is sent after commit.

## 7. Talent Account Lifecycle

```text
PENDING_INVITATION → ACTIVE → DEACTIVATED
```

- Pending accounts cannot use the Talent Portal.
- Active accounts can use the Talent Portal.
- Deactivated accounts cannot log in or use existing sessions.
- Deactivation automatically sets Profile visibility to `HIDDEN`.
- Reactivation does not republish the Profile.

## 8. Profile Visibility Rules

Profile visibility is independent of account status:

```text
HIDDEN ↔ VISIBLE
```

- Admin may edit while hidden.
- Admin cannot edit any Profile field while visible.
- Admin must hide before making any edit.
- Publishing requires all client-visible fields to be complete and valid.
- Client APIs return only active-account, visible, complete Profiles.

Talent may edit permitted fields while visible. Those changes publish immediately,
but the API cannot allow Talent to make a required client-visible value invalid.

## 9. Permanent Verification

Profile creation sets `isVerified = true`. This value is read-only and has no
Admin transition. Visibility is the control for Client exposure.

## 10. Talent Portal Scope

The Portal route is `/talent-portal` with no Profile or Portal ID.

The authenticated User resolves its own Profile through TalentAccount. Talent can
edit only permitted professional fields; Admin-only fields remain protected.

## 11. Out of Scope

- Profile operational status such as Active or Archived.
- Verification workflow after Profile creation.
- Availability, earliest start date, or preferred engagement.
- Assessment data entities.
- Stage-change history.
- Candidate account registration.
- Talent Dashboard.
