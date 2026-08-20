# BlihOps V1 Talent Application and Profile — Data Model

## 1. Purpose

This document defines the planned Prisma model for Talent Applications, Profile
Completion Requests, Talent Profiles, Talent Accounts, and Talent invitations.
It follows the existing PostgreSQL Prisma conventions.

## 2. Enums

```prisma
enum TalentApplicationStatus {
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
}

enum ProfileCompletionRequestStatus {
  PENDING
  SUBMITTED
  EXPIRED
  REPLACED
}

enum InvitationStatus {
  PENDING
  ACCEPTED
  EXPIRED
  REPLACED
}

enum TalentAccountStatus {
  PENDING_INVITATION
  ACTIVE
  DEACTIVATED
}

enum TalentProfileVisibility {
  HIDDEN
  VISIBLE
}
```

There is no Talent Profile lifecycle enum and no verification enum. Verification
is a read-only `isVerified` boolean set to true at Profile creation.

## 3. Existing User Relation

The existing Better Auth `User` model gains an optional Talent Account relation:

```prisma
model User {
  // existing fields...
  talentAccount TalentAccount?
}
```

The service enforces `User.role = TALENT` when creating a Talent Account.

## 4. TalentApplication

```prisma
model TalentApplication {
  id                    String                  @id @default(cuid())
  status                TalentApplicationStatus @default(NEW)
  fullName              String
  workEmail             String
  phone                 String
  country               String
  city                  String
  primaryRole           String
  techStack             String[]
  secondarySkills       String[]
  yearsExperience       Int
  portfolioUrl          String?
  githubUrl             String?
  linkedinUrl           String?
  resumeFileKey               String
  completionPhotoKey          String?
  completionShortBio          String?
  completionProfessionalHeadline String?
  completionSubmittedAt       DateTime?
  internalNotes         String                  @default("")
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt

  completionRequests    ProfileCompletionRequest[]
  talentProfile         TalentProfile?

  @@index([status])
  @@index([workEmail])
  @@index([createdAt])
}
```

Notes:

- `workEmail` should be normalized before persistence and indexed for lookup.
- `techStack` and `secondarySkills` use PostgreSQL string arrays.
- `yearsExperience` is stored as an integer even if the current form sends a
  string; the API parses and validates it.
- `internalNotes` is Admin-only and may be an empty string at database level, but
  the Profile creation form requires Admin to provide the required internal
  creation note defined by the product rules.
- No screening-response JSON field exists.
- No availability, start-date, or engagement fields exist.
- Original Application fields are not updated after submission.

## 5. ProfileCompletionRequest

```prisma
model ProfileCompletionRequest {
  id             String                         @id @default(cuid())
  applicationId  String
  status         ProfileCompletionRequestStatus @default(PENDING)
  tokenHash      String?
  sentAt         DateTime                       @default(now())
  expiresAt      DateTime
  submittedAt    DateTime?
  replacedAt     DateTime?
  createdAt      DateTime                       @default(now())
  updatedAt      DateTime                       @updatedAt

  application    TalentApplication              @relation(fields: [applicationId], references: [id], onDelete: Cascade)

  @@index([applicationId, status])
  @@index([expiresAt])
}
```

The raw token is never stored. If Better Auth or the application owns token
validation, this record stores only a lifecycle reference or hash.

A replacement request creates a new request record and marks the previous one
`REPLACED`. After successful submission, no new correction request is sent.

## 6. TalentProfile

```prisma
model TalentProfile {
  id                    String                  @id @default(cuid())
  applicationId         String                  @unique
  fullName              String
  workEmail             String
  phone                 String
  country               String
  city                  String
  profilePhotoKey       String
  professionalHeadline  String
  shortBio              String
  primaryRole           String
  techStack             String[]
  secondarySkills       String[]
  yearsExperience       Int
  portfolioUrl          String?
  githubUrl             String?
  linkedinUrl           String?
  resumeFileKey         String
  seniority             String
  englishLevel          String
  clientMonthlyRateEur  Decimal               @db.Decimal(12, 2)
  assessmentSummary     String
  internalNotes         String
  isVerified            Boolean               @default(true)
  visibility            TalentProfileVisibility @default(HIDDEN)
  createdAt             DateTime              @default(now())
  updatedAt             DateTime              @updatedAt

  application                 TalentApplication             @relation(fields: [applicationId], references: [id], onDelete: Restrict)
  talentAccount               TalentAccount?
  clientPodMembers            ClientPodMember[]
  pilotRequestParticipants    PilotRequestParticipant[]
  pilotParticipants           PilotParticipant[]
  interviewRequests           InterviewRequest[]

  @@index([visibility])
  @@index([primaryRole])
  @@index([seniority])
  @@index([englishLevel])
  @@index([createdAt])
}
```

There is no operational Profile status. Visibility and Talent Account status
control exposure and access.

`isVerified` is created as true and must not have an update path.

## 7. TalentAccount

```prisma
model TalentAccount {
  id              String             @id @default(cuid())
  userId          String             @unique
  talentProfileId String             @unique
  status          TalentAccountStatus @default(PENDING_INVITATION)
  invitedAt       DateTime?
  activatedAt     DateTime?
  deactivatedAt   DateTime?
  lastLoginAt     DateTime?
  createdAt       DateTime           @default(now())
  updatedAt       DateTime           @updatedAt

  user            User               @relation(fields: [userId], references: [id], onDelete: Cascade)
  talentProfile   TalentProfile      @relation(fields: [talentProfileId], references: [id], onDelete: Restrict)
  invitations     TalentInvitation[]

  @@index([status])
}
```

One Profile has at most one Talent Account. One User has at most one Talent
Account.

## 8. TalentInvitation

```prisma
model TalentInvitation {
  id              String                   @id @default(cuid())
  talentAccountId String
  status          InvitationStatus        @default(PENDING)
  sentAt          DateTime                 @default(now())
  expiresAt       DateTime
  acceptedAt      DateTime?
  replacedAt      DateTime?
  createdAt       DateTime                 @default(now())
  updatedAt       DateTime                 @updatedAt

  talentAccount   TalentAccount            @relation(fields: [talentAccountId], references: [id], onDelete: Cascade)

  @@index([talentAccountId, status])
  @@index([expiresAt])
}
```

If the Client/Company feature is implemented first, reuse this neutral shared
`InvitationStatus` enum for both Client and Talent invitations.

## 9. Field Ownership Map

| Field group | Source | Admin | Talent | Client |
|---|---|---:|---:|---:|
| Original Application data | Application | Read-only | Not directly | No |
| Profile photo and short bio | Completion/Profile | Edit while hidden | Edit after activation | Yes when visible |
| Professional Profile fields | Application/Profile | Edit while hidden | Edit after activation | Yes when visible |
| Seniority | Admin | Edit while hidden | No | Yes when visible |
| English level | Admin | Edit while hidden | No | Yes when visible |
| Client monthly rate | Admin | Edit while hidden | No | Yes when visible |
| Assessment summary | Admin | Edit while hidden | No | No |
| Internal notes | Admin | Edit | No | No |
| Verification | System | Read-only | No | Yes as verified indicator |
| Visibility | Admin | Edit | No | Controls exposure |
| Account status | Admin/system | Edit via action | No | Not exposed |

## 10. Visibility Enforcement

A Profile may be `VISIBLE` only if all client-visible fields are present and
valid, including:

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
- Verification
- Client monthly rate

The API must enforce this validator on the publish operation.

Admin edits are rejected while visible. Admin must hide the Profile before any
edit. Talent updates validate required values and remain allowed while visible.

## 11. Deactivation Enforcement

When `TalentAccount.status` changes to `DEACTIVATED`:

- Revoke/invalidate Talent sessions.
- Set Profile visibility to `HIDDEN` in the same transaction.
- Exclude the Profile from Client-safe queries.

Reactivation sets only the account status to `ACTIVE`; it does not republish the
Profile.

## 12. Storage and Files

- Resume is a required private file reference.
- Profile photo is a required Profile file reference.
- Private files require Talent/Admin authorization.
- Client serialization must use protected or authorized file URLs.
- Replacing a file must not delete the previous file until the new upload is
  committed successfully.

## 13. Indexes and Constraints

- Unique Application-to-Profile relation.
- Unique User-to-TalentAccount relation.
- Unique Profile-to-TalentAccount relation.
- Status indexes for Application and TalentAccount.
- Visibility and directory filter indexes for TalentProfile.
- Email lookup index for Applications.
- No status history table.
- No assessment table.
