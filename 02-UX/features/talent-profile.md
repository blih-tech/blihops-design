# BlihOps V1 Talent Profile and Talent Portal Feature

## 1. Purpose

A Talent Profile is an Admin-created, client-safe representation of an approved
Talent Application. It is created only after the candidate submits the Profile
Completion Request.

The Talent Portal is a single authenticated Profile Management page. It is
resolved from the authenticated Talent Account and does not use a Profile ID in
the URL.

## 2. Profile Creation Flow

```text
Application = COMPLETION_SUBMITTED
→ Admin reviews combined Round 1 + Round 2 data
→ Admin completes required Admin Profile fields
→ Admin creates Talent Profile
→ Create Talent User(role=TALENT)
→ Create Talent Account(PENDING_INVITATION)
→ Profile visibility=HIDDEN
→ Verification=VERIFIED
→ Application=PROFILE_CREATED
→ Send Talent invitation
```

Profile creation, account creation, invitation context, and Application status
change are one transaction. If creation fails, no partial Profile or Account is
left behind.

## 3. Talent Account Status

The Talent Account has the only Talent account lifecycle:

```text
PENDING_INVITATION → ACTIVE → DEACTIVATED
```

| Status | Meaning |
|---|---|
| `PENDING_INVITATION` | Profile and account exist; Talent has not activated access |
| `ACTIVE` | Talent can enter the Talent Portal and edit permitted fields |
| `DEACTIVATED` | Admin revoked access; login and Portal access are blocked |

There is no Talent Profile operational status such as `ACTIVE` or `ARCHIVED`.
The Profile remains an Admin-owned historical record.

## 4. Profile Visibility

Visibility is separate from account status:

```text
HIDDEN ↔ VISIBLE
```

### HIDDEN

- Talent may still use the Portal if the account is active.
- Clients cannot discover or open the Profile.
- Admin may edit the Profile.
- Admin may complete the publish checklist.

### VISIBLE

- Clients may discover the Profile if the Account is active and all required
  client-visible fields remain valid.
- Admin cannot edit any Profile field while it is visible.
- Admin must first change visibility to `HIDDEN` before editing.
- Talent may continue editing permitted fields immediately.

### Deactivation

When Admin deactivates the Talent Account:

```text
Talent Account: ACTIVE → DEACTIVATED
Profile visibility: automatically → HIDDEN
```

The Profile remains in Admin history. Reactivation changes only the account:

```text
Talent Account: DEACTIVATED → ACTIVE
Profile visibility: remains HIDDEN
```

Admin must explicitly publish it again.

## 5. Verification

Every Profile created from a completed, approved Application is permanently
verified:

```text
isVerified = true
```

Verification:

- Is set automatically during Profile creation.
- Cannot be changed by Admin.
- Cannot be changed by Talent.
- Can be displayed to Clients as a verified indicator.

If Admin no longer wants the Profile exposed, Admin uses visibility, not
verification.

## 6. Profile Fields and Ownership

### 6.1 Copied from Application and Completion Data

- Full name
- Work email
- Phone
- Country
- City
- Profile photo
- Professional headline
- Short bio
- Primary role
- Tech stack
- Secondary skills
- Years of experience
- Portfolio
- GitHub
- LinkedIn
- Resume

The original Application remains immutable. These values are copied into the
Profile at creation.

### 6.2 Required Admin Creation Fields

Admin must provide all of the following before Profile creation:

- Seniority
- English level
- Client monthly selling rate
- Assessment summary
- Internal notes

### 6.3 Admin-only Fields

Admin controls:

- Seniority
- English level
- Client monthly selling rate
- Assessment summary
- Internal notes
- Visibility
- Talent Account status
- Verification, which is read-only after creation

### 6.4 Talent-editable Fields

After activation, Talent may edit:

- Profile photo
- Professional headline
- Short bio
- Primary role
- Tech stack
- Secondary skills
- Years of experience
- Portfolio
- GitHub
- LinkedIn
- Resume

Talent-originated changes publish immediately. No Admin approval is required.

The API must still validate that Talent updates cannot clear required client-visible
values or change Admin-only fields.

### 6.5 Fields Removed from V1

The following do not exist in Round 1, Round 2, Profile creation, or Talent Portal:

- Screening responses
- Availability
- Earliest start date
- Preferred engagement

## 7. Client Visibility Checklist

Admin can change:

```text
HIDDEN → VISIBLE
```

only when all client-visible fields are present and valid:

- Profile photo
- Full name
- Professional headline
- Short bio
- Primary role
- Seniority
- Years of experience
- Tech stack
- Secondary skills
- Portfolio
- GitHub
- LinkedIn
- Resume
- English level
- Verification indicator
- Client monthly selling rate

The API performs this validation independently of the Admin UI. The Admin UI
shows a blocking checklist with missing or invalid values.

A visible Profile cannot be edited by Admin. Admin must hide it before any edit,
then publish again after validation.

## 8. Talent Invitation and Portal

After Profile creation:

```text
Talent Account = PENDING_INVITATION
```

The invitation is sent automatically. After acceptance:

```text
Talent Account = ACTIVE
→ /talent-portal
```

The API resolves the Profile through:

```text
Authenticated User
→ Talent Account
→ Talent Profile
```

There is no `/talent-portal/:profileId` or `/talent-portal/:portalId` route.

## 9. Talent Portal Experience

The Portal contains one focused Profile Management page:

- Profile header and photo
- Professional information
- Skills and experience
- Portfolio/social links
- Resume
- Short bio
- Save feedback
- Account actions

It does not contain:

- Dashboard
- Talent discovery
- Client information
- Rates
- Assessment information
- Verification controls
- Visibility controls
- Account-status controls
- Availability controls

## 10. Admin Profile Experience

### Talent Profiles list

Displays:

- Talent name
- Primary role
- Seniority
- Account status
- Visibility
- Client monthly selling rate
- Verification indicator
- Created date

### Talent Profile Detail

Contains:

- Client-safe Profile data
- Admin-managed fields
- Account and invitation state
- Visibility checklist
- Linked Application
- Admin-only internal notes
- Talent-originated update indication

Admin can:

- View Profile data.
- Edit only while hidden.
- Publish after checklist validation.
- Hide a visible Profile.
- Deactivate/reactivate Talent Account.
- Resend invitation.
- Open the source Application.

## 11. Out of Scope

- Talent Profile operational archive status
- Talent verification editing after creation
- Talent availability management
- Preferred engagement or start-date management
- Client editing of Profiles
- Client-to-Talent messaging
- Talent-created initial Profiles
- Talent Dashboard
