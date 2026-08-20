# BlihOps V1 Talent Application and Profile — Implementation Plan

## 1. Implementation Order

```text
BlihOps API
→ BlihOps Admin
→ blihops-web
→ Client/Company integration
→ End-to-end verification
```

Talent Profiles must be implemented first because Client Talent Directory, Pods,
Interview Requests, and Pilot participants depend on the Profile contract.

## 2. Phase 0 — Preparation

- [ ] Confirm the Talent field contract in the UX and data-model documents.
- [ ] Remove obsolete `screening`, `availability`, `startDate`, and `engagement`
      fields from Web schemas and API payloads.
- [ ] Confirm `professionalHeadline` stays in Round 2 (completion form).
- [ ] Confirm the existing auth invitation and reset-token integration points.
- [ ] Confirm protected storage helpers for resume and profile photo.
- [ ] Do not create a Profile status or `/talent-portal/:id` route.

## 3. Phase 1 — Blihops API

### 3.1 Prisma schema and migration

File:

```text
blihops-api/prisma/schema.prisma
```

Add:

- `TalentApplication`
- `ProfileCompletionRequest`
- `TalentProfile`
- `TalentAccount`
- `TalentInvitation`
- Application, completion, account, and visibility enums.

Add User relations and any required file/reference fields.

Rules:

- Original Application fields are immutable after creation.
- One Profile per Application.
- One Talent Account per Profile and User.
- `isVerified` defaults to true and has no update path.
- Profile visibility defaults to hidden.
- No Profile operational status.

Run the repository's Prisma format/generate/migrate commands and inspect the
migration before applying it.

### 3.2 API feature modules

Recommended structure:

```text
blihops-api/src/features/talent/
├── index.ts
├── applications/
├── completion/
├── profiles/
├── accounts/
└── shared/
```

Mirror the existing feature conventions used by `auth`, `leads`, and content:
controller, repository, router, schema, service, and paths where applicable.

### 3.3 Public Application endpoint

Implement:

```text
POST /api/v1/talent-applications
```

- Validate Round 1 fields.
- Require resume.
- Reject obsolete fields.
- Store normalized values and protected resume reference.
- Set status `NEW`.
- Return confirmation without internal fields.

### 3.4 Admin Application endpoints

Implement:

```text
GET   /api/v1/admin/talent-applications
POST  /api/v1/admin/talent-applications/:applicationId/status
PATCH /api/v1/admin/talent-applications/:applicationId/notes
GET   /api/v1/admin/talent-applications/:applicationId
POST  /api/v1/admin/talent-applications/:applicationId/completion-request
```

- Support search, filters, pagination, and detail projections.
- Allow flexible status changes.
- Do not require status reason.
- Do not create status history records.
- Keep Application fields immutable.
- Return internal notes only to Admin.
- Allow completion request only after `APPROVED`.

### 3.5 Completion request endpoints

Implement:

```text
GET  /api/v1/profile-completion-requests/:token
POST /api/v1/profile-completion-requests/:token/submit
```

- Token is single-use and expires after seven days.
- Form accepts only photo and short bio.
- Successful submission updates Application completion fields and status.
- No Profile/account is created at submission time.
- Expired pending requests can be replaced by Admin.
- No correction loop after successful submission.

### 3.6 Profile creation endpoint

Implement:

```text
POST /api/v1/admin/talent-applications/:applicationId/create-profile
```

Require:

```text
Application.status = COMPLETION_SUBMITTED
```

Require all Admin creation fields:

- Seniority
- English level
- Client monthly selling rate
- Assessment summary
- Internal notes

Transaction:

```text
Create Profile
→ isVerified=true
→ visibility=HIDDEN
→ Create User(role=TALENT)
→ Create TalentAccount(PENDING_INVITATION)
→ Create invitation context
→ Application=PROFILE_CREATED
→ Commit
→ Send invitation
```

### 3.7 Talent Account and Portal endpoints

Implement:

```text
GET   /api/v1/talent/me
GET   /api/v1/talent/profile
PATCH /api/v1/talent/profile
POST  /api/v1/talent/profile/photo
POST  /api/v1/talent/profile/resume
```

- Resolve Profile through session User → TalentAccount.
- Reject Profile IDs and cross-Talent access.
- Accept only Talent-editable fields.
- Publish valid Talent edits immediately.
- Reject Admin-only fields and invalid required values.

### 3.8 Admin Profile endpoints

Implement:

```text
GET   /api/v1/admin/talent-profiles
GET   /api/v1/admin/talent-profiles/:profileId
PATCH /api/v1/admin/talent-profiles/:profileId
POST  /api/v1/admin/talent-profiles/:profileId/show
POST  /api/v1/admin/talent-profiles/:profileId/hide
POST  /api/v1/admin/talent-profiles/:profileId/deactivate-account
POST  /api/v1/admin/talent-profiles/:profileId/reactivate-account
POST  /api/v1/admin/talent-profiles/:profileId/invitation
```

Rules:

- Admin Profile edits are rejected while visible.
- `show` runs the complete client-visible checklist.
- `hide` is always allowed to Admin.
- Deactivation hides Profile and revokes access atomically.
- Reactivation leaves Profile hidden.
- Verification is never editable.

### 3.9 Client-safe Profile endpoint

Implement or finalize:

```text
GET /api/v1/client/talents
GET /api/v1/client/talents/:profileId
```

Return only active, visible, complete Profiles and safe fields. This contract is
needed by the later Client/Company implementation.

### 3.10 API tests

Cover:

- Round 1 submission and obsolete-field rejection.
- Required resume/photo behavior.
- Flexible statuses and rejected reopening.
- Immutable Application fields.
- Admin-only notes.
- Completion token expiry/consumption/replacement.
- Profile creation prerequisites and transaction rollback.
- Automatic invitation creation.
- Talent self-access and field permissions.
- Immediate Talent publishing.
- Admin visible-Profile edit blocking.
- Visibility checklist.
- Deactivation auto-hide and reactivation behavior.
- Client-safe Profile filtering.

Run:

```bash
cd blihops-api
pnpm lint
pnpm typecheck
pnpm test:all
pnpm format:check
pnpm build
```

## 4. Phase 2 — BlihOps Admin

### 4.1 Replace Talent Applications placeholder

Current route:

```text
blihops-admin/src/app/(dashboard)/talent/applications/page.tsx
```

Implement:

- Applications table.
- Search/filter/pagination.
- Status badges.
- Completion state.
- Profile-created state.
- Loading, empty, error, and retry states.

### 4.2 Application Detail

Display:

- Immutable Round 1 data.
- Resume access.
- Completion data.
- Internal notes field.
- Flexible status control.
- Completion request action.
- Create Talent Profile action when eligible.

Do not provide edit controls for original submission fields.

### 4.3 Completion request actions

Admin can:

- Send request after approval.
- Resend/replace expired pending request.
- See request status.
- See submission status.

After submission, no correction-request action is needed.

### 4.4 Create Profile form

The form shows imported data as read-only and requires:

- Seniority.
- English level.
- Client monthly selling rate.
- Assessment summary.
- Internal notes.

On success:

- Application becomes Profile Created.
- Profile starts hidden and verified.
- Talent account is pending invitation.
- Invitation state is visible.
- Profile Detail becomes available.

### 4.5 Replace Talent Profiles placeholder

Current route:

```text
blihops-admin/src/app/(dashboard)/talent/profiles/page.tsx
```

Implement:

- Profile list.
- Filters for visibility, account status, role, seniority.
- Search and pagination.
- Visibility/account status columns.
- Link to Profile Detail.

### 4.6 Profile Detail

Display:

- Profile fields.
- Admin fields.
- Account/invitation state.
- Visibility checklist.
- Linked Application.
- Internal notes.
- Talent-originated update indication.

Actions:

- Hide visible Profile.
- Edit hidden Profile.
- Publish after checklist passes.
- Deactivate/reactivate Talent Account.
- Resend invitation.
- Open source Application.

### 4.7 Admin verification

Run:

```bash
cd blihops-admin
pnpm lint
pnpm typecheck
pnpm format:check
pnpm build
```

## 5. Phase 3 — blihops-web

### 5.1 Update Round 1 form

Current files:

```text
blihops-web/src/app/[locale]/talent/apply/
blihops-web/src/components/sections/talent-apply/TalentApplicationForm.tsx
blihops-web/src/lib/forms/talent-application.ts
```

Implement:

- Add country.
- Remove screening responses.
- Remove professional headline (it is collected in Round 2).
- Preserve required resume upload.
- Match API field names and types.
- Keep optional URLs consistent with the visibility contract.

### 5.2 Update Round 2 form

Current files:

```text
blihops-web/src/app/[locale]/talent/complete-profile/
blihops-web/src/components/sections/talent-complete-profile/CompleteProfileForm.tsx
blihops-web/src/lib/forms/talent-complete-profile.ts
```

Keep:

- Profile photo.
- Short bio.
- Professional headline.

Remove:

- Availability.
- Earliest start date.
- Preferred engagement.

### 5.3 Replace Talent Portal route

Current scaffold:

```text
blihops-web/src/app/[locale]/talent-portal/[portalId]/
```

Replace with:

```text
blihops-web/src/app/[locale]/talent-portal/
```

The page and layout must:

- Require an authenticated Talent.
- Resolve the Profile through the session.
- Redirect non-Talent roles safely.
- Show no Portal/Profile ID.
- Show account-deactivated state.

### 5.4 Implement Talent Profile Management

Sections:

- Profile photo.
- Short bio.
- Professional headline.
- Primary role.
- Tech stack and secondary skills.
- Years of experience.
- Portfolio/GitHub/LinkedIn.
- Resume.

Do not show or allow edits for:

- Rates.
- Seniority.
- English level.
- Verification.
- Visibility.
- Assessment summary.
- Internal notes.
- Account status.
- Availability.
- Earliest start date.
- Preferred engagement.

Successful saves should update the Profile immediately.

### 5.5 Web verification

Run:

```bash
cd blihops-web
pnpm lint
pnpm typecheck
pnpm format:check
pnpm build
```

Verify:

- Application submission uses updated fields.
- Completion form contains only photo and bio.
- Talent invitation redirects to `/talent-portal`.
- Talent cannot access another Profile.
- Talent updates publish immediately.
- Admin-only fields are never exposed in the Portal.

## 6. Phase 4 — Client/Company Integration

After Profile endpoints are stable:

- Replace dummy Talent directory data with Client-safe Profile API data.
- Remove availability from Client Talent cards and Detail.
- Use the final Profile ID for Pod membership.
- Use the final Profile ID for Interview Requests.
- Use the final Profile ID for Pilot participants.
- Exclude hidden/deactivated Profiles from Client discovery.
- Preserve unavailable references where historical Client operations require them.

## 7. Phase 5 — End-to-End Verification

Test:

```text
Candidate applies
→ Admin reviews and changes statuses
→ Admin approves
→ Completion request sent
→ Candidate submits photo + bio
→ Admin creates Profile
→ Talent invitation sent
→ Talent activates account
→ Talent enters /talent-portal
→ Talent edits permitted fields
→ Admin publishes Profile
→ Client sees Profile
```

Also test:

```text
Visible Profile
→ Admin edit attempt rejected
→ Admin hides Profile
→ Admin edits Profile
→ Admin publishes after checklist
```

And:

```text
Active Talent Account
→ Admin deactivates
→ Session/access revoked
→ Profile hidden
→ Admin reactivates
→ Account active but Profile remains hidden
```

## 8. Suggested Commit Sequence

```text
feat(api): add talent application and profile schema
feat(api): implement public talent applications
feat(api): implement application completion requests
feat(api): implement talent profile creation and invitations
feat(api): implement talent account and portal APIs
feat(api): implement profile visibility and client-safe reads
feat(admin): implement talent applications list and detail
feat(admin): implement completion and profile creation flows
feat(admin): implement talent profiles and visibility management
feat(web): update talent application form
feat(web): update profile completion form
feat(web): replace portal id route with talent portal
feat(web): implement talent profile management
feat(client): connect talent directory to profile API
 test: verify talent lifecycle end to end
```

## 9. Definition of Done

- Round 1 contains no screening field.
- Round 2 contains only photo and short bio.
- No availability, start-date, or engagement fields exist.
- Application fields are immutable.
- Admin can move Application statuses flexibly.
- Rejected Applications can be reopened.
- No status-change history or assessment records are created.
- Admin-only internal notes work.
- Profile creation requires completion data and all Admin fields.
- Profile creation automatically creates a pending Talent Account and sends an
  invitation.
- Verification is permanently set to true.
- Profile starts hidden.
- Admin cannot edit visible Profiles.
- Admin publish validation blocks incomplete Profiles.
- Deactivation hides Profile and blocks Talent access.
- Reactivation leaves Profile hidden.
- Talent Portal uses `/talent-portal` without an ID.
- Talent edits publish immediately.
- Client APIs expose only active, visible, complete Profiles.
- API, Admin, and Web checks pass.
- Client/Company can consume the final Talent Profile contract.
