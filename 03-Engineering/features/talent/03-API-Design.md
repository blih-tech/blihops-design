# BlihOps V1 Talent Application and Profile — API Design

## 1. Purpose

This document defines the API contract for the public Talent Application,
Profile Completion Request, Admin recruitment operations, Talent Portal, and
Client-safe Talent Profile reads.

All authorization and Profile visibility rules are enforced server-side.

## 2. Roles and Ownership

### Public

Can submit a Talent Application and receive a confirmation. Public requests
cannot choose an Application status or create an account.

### Admin

Can review Applications, move statuses flexibly, manage internal notes, issue
completion requests, create Profiles, manage visibility, and manage Talent
Account access.

### Talent

Can access only its own Profile through a session-derived TalentAccount and edit
permitted fields after activation.

### Client

Can only read Profiles that pass the Client visibility query. Client endpoints
never expose Admin fields or private Application data.

## 3. Route Inventory

### 3.1 Public Application

```text
POST /api/v1/talent-applications
```

Creates a new Application with status `NEW`.

The server sets the status. The payload cannot include status, role, account, or
Profile fields.

### 3.2 Profile Completion Request

```text
GET  /api/v1/profile-completion-requests/:token
POST /api/v1/profile-completion-requests/:token/submit
```

The token identifies one pending request. The response contains only the fields
needed to display the completion form. Submission accepts:

```json
{
  "photoFileKey": "private/uploads/photo.webp",
  "shortBio": "A concise professional biography.",
  "professionalHeadline": "Senior backend engineer"
}
```

Successful submission:

- Marks the request `SUBMITTED`.
- Updates Application completion fields.
- Sets Application status `COMPLETION_SUBMITTED`.
- Consumes the token.
- Does not create a Profile or account.

### 3.3 Admin Applications

```text
GET   /api/v1/admin/talent-applications
POST  /api/v1/admin/talent-applications
GET   /api/v1/admin/talent-applications/:applicationId
PATCH /api/v1/admin/talent-applications/:applicationId/status
PATCH /api/v1/admin/talent-applications/:applicationId/notes
POST  /api/v1/admin/talent-applications/:applicationId/completion-request
```

`POST` manual Application creation is optional if the Admin product requires it;
the public flow remains the primary creation path.

Status mutation accepts a target status from the allowed enum. No reason is
required and no status history record is created.

Status endpoint must still reject impossible system states:

- Completion request unless Application is `APPROVED`.
- Profile creation unless Application is `COMPLETION_SUBMITTED`.

Notes endpoint updates only the Admin-only internal notes field.

Completion request endpoint:

- Creates a new pending request or replaces an expired request.
- Sends the essential completion email.
- Does not create a Profile or account.
- Does not expose the raw token in the Admin response.

### 3.4 Admin Talent Profiles

```text
GET   /api/v1/admin/talent-profiles
GET   /api/v1/admin/talent-profiles/:profileId
POST  /api/v1/admin/talent-applications/:applicationId/create-profile
PATCH /api/v1/admin/talent-profiles/:profileId
POST  /api/v1/admin/talent-profiles/:profileId/show
POST  /api/v1/admin/talent-profiles/:profileId/hide
POST  /api/v1/admin/talent-profiles/:profileId/deactivate-account
POST  /api/v1/admin/talent-profiles/:profileId/reactivate-account
POST  /api/v1/admin/talent-profiles/:profileId/invitation
```

`create-profile` accepts the required Admin fields:

```json
{
  "seniority": "Senior",
  "englishLevel": "C1",
  "clientMonthlyRateEur": "4200.00",
  "assessmentSummary": "Passed all required review stages.",
  "internalNotes": "Approved for client visibility after final review."
}
```

The endpoint copies immutable Application and completion data, creates the
Profile/User/TalentAccount/invitation context transactionally, sets
`isVerified=true`, sets visibility `HIDDEN`, and changes the Application status
to `PROFILE_CREATED`.

`PATCH /admin/talent-profiles/:profileId` must reject all edits while visibility
is `VISIBLE`. Admin must hide first.

`show` runs the complete Client visibility validator. It returns missing/invalid
fields if publishing is blocked.

`deactivate-account` sets the Talent Account to `DEACTIVATED`, revokes access,
and hides the Profile atomically.

`reactivate-account` sets the Account to `ACTIVE` but leaves the Profile hidden.

### 3.5 Talent Portal

```text
GET   /api/v1/talent/me
GET   /api/v1/talent/profile
PATCH /api/v1/talent/profile
POST  /api/v1/talent/profile/photo
POST  /api/v1/talent/profile/resume
```

The API resolves the Talent Profile from:

```text
session.user.id
→ TalentAccount.userId
→ TalentProfile
```

There is no Profile ID or Portal ID in the Talent route or authorization model.

`PATCH /talent/profile` accepts only Talent-editable fields. It rejects:

- Admin-only fields.
- Account status.
- Visibility.
- Verification.
- Internal notes.
- Any update that would make required Client-visible values invalid.

Successful Talent updates publish immediately and record the origin as Talent.

### 3.6 Client-safe Talent Profiles

```text
GET /api/v1/client/talents
GET /api/v1/client/talents/:profileId
```

The query requires:

```text
TalentAccount.status = ACTIVE
TalentProfile.visibility = VISIBLE
Client-visible completeness validator = PASS
```

The serializer returns only:

- Photo
- Name
- Headline
- Short bio
- Role
- Seniority
- Experience
- Tech stack and skills
- Portfolio, GitHub, LinkedIn
- Resume access
- English level
- Verification indicator
- Client monthly rate

It never returns:

- Internal notes
- Assessment summary
- Application data not intended for Clients
- Account status
- Invitation state
- Private file metadata

## 4. Payload Contracts

### 4.1 Public Application

```json
{
  "fullName": "Example Candidate",
  "workEmail": "candidate@example.com",
  "phone": "+251900000000",
  "country": "Ethiopia",
  "city": "Addis Ababa",
  "primaryRole": "Backend Engineer",
  "techStack": ["TypeScript", "Node.js", "PostgreSQL"],
  "secondarySkills": ["Docker", "AWS"],
  "yearsExperience": 7,
  "portfolioUrl": "https://example.com",
  "githubUrl": "https://github.com/example",
  "linkedinUrl": "https://linkedin.com/in/example",
  "resumeFileKey": "private/resumes/example.pdf"
}
```

No `screening`, `availability`, `startDate`, or `engagement` fields are accepted.

### 4.2 Talent Self-Update

```json
{
  "professionalHeadline": "Senior backend engineer",
  "shortBio": "A concise professional biography.",
  "primaryRole": "Backend Engineer",
  "techStack": ["TypeScript", "Node.js"],
  "secondarySkills": ["Docker"],
  "yearsExperience": 7,
  "portfolioUrl": "https://example.com",
  "githubUrl": "https://github.com/example",
  "linkedinUrl": "https://linkedin.com/in/example"
}
```

Admin-only and account fields are rejected, not silently ignored.

## 5. Visibility Validation

The server returns a structured blocking response for failed publishing:

```json
{
  "code": "PROFILE_NOT_READY_FOR_VISIBILITY",
  "missingFields": [
    "clientMonthlyRateEur",
    "englishLevel"
  ]
}
```

The exact user-facing validation copy is an implementation concern, but the API
must expose stable field identifiers.

## 6. Error Codes

```text
APPLICATION_NOT_FOUND
APPLICATION_STATUS_INVALID
APPLICATION_FIELDS_IMMUTABLE
COMPLETION_REQUEST_NOT_ALLOWED
COMPLETION_TOKEN_INVALID
COMPLETION_TOKEN_EXPIRED
COMPLETION_TOKEN_CONSUMED
PROFILE_CREATION_NOT_ALLOWED
PROFILE_ALREADY_EXISTS
PROFILE_NOT_FOUND
PROFILE_VISIBLE_READ_ONLY
PROFILE_NOT_READY_FOR_VISIBILITY
TALENT_ACCOUNT_INACTIVE
TALENT_PROFILE_NOT_OWNED
TALENT_FIELD_FORBIDDEN
TALENT_PROFILE_INVALID_UPDATE
TALENT_PROFILE_HIDDEN
```

## 7. File Rules

- Resume is required during Application submission.
- Profile photo is required during completion submission.
- Private candidate files require authorized access.
- Profile photo/resume replacement must be transactional from the user's point
  of view.
- File keys and raw invitation tokens are not exposed unnecessarily.

## 8. No General Notifications

Essential email is limited to:

- Profile Completion Request.
- Talent Account invitation.
- Password reset.

Application stage changes, profile changes, visibility changes, and Talent edits
do not generate automated status emails in V1.
