# BlihOps V1 Talent Application Feature

## 1. Purpose

The Talent Application is the public recruitment entry point for candidates who
want to join the Blihops Talent Pool. It collects the information Admin needs
to review and assess a candidate, then supports a second, lightweight completion
request after approval.

The Application is the immutable record of the candidate's original submission.
Admin may add private internal notes and change the current pipeline status, but
cannot edit the submitted Application fields.

## 2. Actors

### Candidate

- Completes the public Talent Application.
- Uploads a required resume.
- After approval, completes the Profile Completion Request with a photo and short
  bio.
- Does not create a Profile or account directly.

### Admin

- Reviews applications.
- Moves applications flexibly through the pipeline.
- Adds private internal notes.
- Approves or rejects candidates.
- Sends the completion request after approval.
- Creates the Talent Profile after completion data is submitted.

## 3. Round 1 Application

The public form is `/talent/apply`.

### Required fields

| Field | Type | Notes |
|---|---|---|
| Full name | String | Candidate identity |
| Work email | Email string | Used for recruitment and later invitation |
| Phone | String | Contact number |
| Country | String | Candidate location |
| City | String | Candidate location |
| Primary role | String/enum | Main professional role |
| Tech stack | String array | At least one item |
| Secondary skills | String array | May be empty if none |
| Years of experience | Integer | Non-negative, bounded value |
| Portfolio | URL string | Optional at submission; required before Client visibility if included in the public Profile contract |
| GitHub | URL string | Optional at submission; required before Client visibility if included in the public Profile contract |
| LinkedIn | URL string | Optional at submission; required before Client visibility if included in the public Profile contract |
| Resume | Private file reference | Required; supported document types and size limits apply |

The following field is intentionally not collected:

- Screening responses

The candidate does not provide availability, earliest start date, or preferred
engagement in V1.

## 4. Application Statuses

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

### Meaning

| Status | Meaning |
|---|---|
| `NEW` | Newly submitted application |
| `UNDER_REVIEW` | Admin is reviewing the submission |
| `SCREENING` | Candidate is in the screening stage |
| `TECHNICAL_ASSESSMENT` | Candidate is in technical evaluation |
| `ENGLISH_ASSESSMENT` | Candidate is in English evaluation |
| `REMOTE_READINESS_ASSESSMENT` | Candidate is in remote-readiness evaluation |
| `APPROVED` | Admin approved the candidate for completion |
| `COMPLETION_REQUESTED` | Admin sent the second-round completion form |
| `COMPLETION_SUBMITTED` | Candidate submitted photo and short bio |
| `PROFILE_CREATED` | Admin created the Talent Profile and Talent Account |
| `REJECTED` | Admin rejected the Application |
| `ARCHIVED` | Admin archived the Application record |

## 5. Flexible Status Transitions

Admin may move an Application directly to any workflow status when operationally
necessary. Admin may also move an Application backward.

```text
Any workflow stage ↔ any other workflow stage
```

`REJECTED` can be reverted to any active workflow stage without creating a new
Application.

There is no required transition reason and no status-transition history record.
The current status is the only pipeline decision stored for the Application.

Hard system gates still apply:

- A completion request can only be sent after `APPROVED`.
- `COMPLETION_SUBMITTED` is produced by the completion form submission.
- A Profile can only be created after completion data has been submitted.
- `PROFILE_CREATED` is set by the Profile creation action.

## 6. Immutable Submission Rules

The following original Application fields cannot be edited after submission:

- Identity and contact fields
- Location
- Professional fields
- Skills
- Experience
- Portfolio and social links
- Resume submission

Admin can instead use the internal notes field for context or corrections.

## 7. Admin Internal Notes

Each Application has one Admin-only internal notes field.

Admin can:

- Add notes at any Application stage.
- Edit the notes later.
- Use the notes for any information related to the candidate.

The notes are never returned to:

- The Candidate
- Talent Portal
- Clients
- Public Web users

Internal notes are separate from status transitions. Status changes do not create
history records or require notes.

## 8. Profile Completion Request

The completion request is available only after the Application reaches `APPROVED`.

```text
APPROVED
→ Admin sends completion request
→ COMPLETION_REQUESTED
→ Candidate opens single-use link
→ Candidate submits profile photo, short bio, and professional headline
→ COMPLETION_SUBMITTED
```

### Round 2 fields

- Profile photo
- Short bio
- Professional headline

The completion request does not collect:

- Screening responses
- Availability
- Earliest start date
- Preferred engagement

The form is unauthenticated and protected by a single-use token. It does not
create a Profile or account.

### Token rules

- Token expires after 7 days.
- Token is single-use.
- Invalid, expired, or consumed tokens cannot open the form.
- Admin may resend or replace an expired pending request.
- After successful submission, no correction loop or second completion request is
  created. Admin creates the Profile, and the Talent can improve permitted fields
  after activation through the Talent Portal.

## 9. Admin Application Experience

### Applications list

Displays:

- Candidate name
- Primary role
- Location
- Years of experience
- Current status
- Completion state
- Profile creation state
- Submitted date

Admin can search, filter by status/role, paginate, and open Application Detail.

### Application Detail

Contains:

- Immutable Round 1 submission data
- Resume preview/download
- Completion Request state
- Submitted Round 2 data
- Internal notes
- Flexible status control
- Create Talent Profile action when completion is submitted

The Create Talent Profile action opens the final Profile creation form and is not
available before `COMPLETION_SUBMITTED`.

## 10. Out of Scope

- Public account creation
- Candidate self-created Talent Profiles
- Public candidate status tracking
- Assessment score records
- Pass/fail assessment records
- Stage-change history
- Automated recruitment messages beyond essential completion and invitation links
- Candidate-to-Admin messaging system
