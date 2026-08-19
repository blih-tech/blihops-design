# BlihOps V1 Client and Company Feature — Data Model

## 1. Purpose

This document defines the planned persistence model for Client Accounts,
Companies, Interview Requests, private Pods, Pilot Requests, Pilots, and
progress updates.

It follows the existing Prisma conventions in `blihops-api/prisma/schema.prisma`.
The existing Better Auth `User`, `Session`, `Account`, and `Verification`
models remain the authentication foundation.

## 2. Modeling Decisions

- `Company` is the business and authorization boundary.
- There is no `ClientWorkspace` model in V1.
- `ClientAccount` connects a Better Auth User to one Company and owns client
  access status.
- One Company has at most one primary ClientAccount in V1.
- Pilot Requests and Pilots are separate records because a request can be
  rejected/resubmitted and a Pilot can be cancelled without consuming the free
  Pilot opportunity.
- A Company may have multiple unsuccessful request/Pilot attempts in history,
  but at most one completed successful Pilot.
- Pods are Company-owned and client-private.
- Pilot and Interview participant membership is snapshotted at operation time.

## 3. Enums

```prisma
enum ClientAccountStatus {
  PENDING_INVITATION
  ACTIVE
  DEACTIVATED
}

enum CompanyStatus {
  PILOT_PREPARATION
  ON_PILOT
  PILOT_COMPLETED
  ACTIVE
  ARCHIVED
}

enum ClientInvitationStatus {
  PENDING
  ACCEPTED
  EXPIRED
  REPLACED
}

enum PilotRequestStatus {
  SUBMITTED
  IN_REVIEW
  REJECTED
  APPROVED
}

enum PilotStatus {
  PLANNED
  ACTIVE
  COMPLETED
  CANCELLED
}

enum InterviewRequestStatus {
  PENDING
  SCHEDULED
  COMPLETED
  REJECTED
  CANCELLED
}

enum InterviewRequestOrigin {
  INDIVIDUAL
  POD
}

enum ParticipantSource {
  INDIVIDUAL
  POD
}

enum PodStatus {
  ACTIVE
  ARCHIVED
}
```

## 4. Better Auth User Relation

The existing `User` model keeps its authentication fields and role field. Add an
optional relation for the client-specific account:

```prisma
model User {
  // existing fields...
  clientAccount ClientAccount?
  pilotUpdates  PilotProgressUpdate[] @relation("PilotUpdateAuthor")
}
```

The role remains `CLIENT` for this feature. A Client password is created only
through the invitation acceptance flow.

## 5. ClientAccount

```prisma
model ClientAccount {
  id            String             @id @default(cuid())
  userId        String             @unique
  companyId     String             @unique
  status        ClientAccountStatus @default(PENDING_INVITATION)
  invitedAt     DateTime?
  activatedAt   DateTime?
  deactivatedAt DateTime?
  lastLoginAt   DateTime?
  createdAt     DateTime           @default(now())
  updatedAt     DateTime           @updatedAt

  user          User               @relation(fields: [userId], references: [id], onDelete: Cascade)
  company       Company            @relation(fields: [companyId], references: [id], onDelete: Restrict)
  invitations   ClientInvitation[]
}
```

`userId @unique` and `companyId @unique` enforce one Client Account per User and
one primary Client Account per Company in V1.

## 6. Company

```prisma
model Company {
  id                String             @id @default(cuid())
  name              String
  website           String?
  industry          String?
  companySize       String?
  country           String?
  notes             String?
  status            CompanyStatus      @default(PILOT_PREPARATION)
  sourceLeadId      String?            @unique
  archivedAt        DateTime?
  archivedReason    String?
  createdAt         DateTime           @default(now())
  updatedAt         DateTime           @updatedAt

  sourceLead        Lead?              @relation(fields: [sourceLeadId], references: [id], onDelete: SetNull)
  clientAccount     ClientAccount?
  pilotRequests     PilotRequest[]
  pilots            Pilot[]
  interviewGroups   InterviewRequestGroup[]
  interviewRequests InterviewRequest[]
  pods              ClientPod[]

  @@index([status])
  @@index([createdAt])
}
```

There is no `workspaceId` or `ClientWorkspace` relation. The Company ID is the
scope used by Client queries.

The existing `Lead` model may add:

```prisma
company Company?
```

A conversion service must enforce duplicate-company policy before setting the
Lead relation.

## 7. Client Invitation

The invitation lifecycle is separate from the Client Account lifecycle.
Better Auth may continue to issue and consume the secure reset-style token;
this record provides feature-level status and audit context.

```prisma
model ClientInvitation {
  id              String                   @id @default(cuid())
  clientAccountId  String
  status          ClientInvitationStatus   @default(PENDING)
  sentAt          DateTime                 @default(now())
  expiresAt       DateTime
  acceptedAt      DateTime?
  replacedAt      DateTime?
  createdAt       DateTime                 @default(now())
  updatedAt       DateTime                 @updatedAt

  clientAccount   ClientAccount            @relation(fields: [clientAccountId], references: [id], onDelete: Cascade)

  @@index([clientAccountId, status])
  @@index([expiresAt])
}
```

Raw invitation tokens must not be stored in this table. If the application owns
token validation rather than Better Auth, store only a cryptographic token hash.

## 8. PilotRequest

```prisma
model PilotRequest {
  id                   String              @id @default(cuid())
  companyId            String
  status               PilotRequestStatus  @default(SUBMITTED)
  title                String
  businessGoals        String
  successMeasurements  String
  expectedStartDate    DateTime
  durationDays         Int                 @default(14)
  additionalContext    String?
  submittedAt          DateTime            @default(now())
  reviewedAt           DateTime?
  rejectionReason      String?
  resubmissionCount    Int                 @default(0)
  createdAt            DateTime            @default(now())
  updatedAt            DateTime            @updatedAt

  company              Company             @relation(fields: [companyId], references: [id], onDelete: Restrict)
  participants         PilotRequestParticipant[]
  pilot                Pilot?

  @@index([companyId, status])
  @@index([submittedAt])
}
```

The service must prevent another non-final request/Pilot from being created for
the same Company. A request can be resubmitted after rejection; its status
history belongs in the activity/audit system.

## 9. Pilot Request Participants

```prisma
model PilotRequestParticipant {
  id              String          @id @default(cuid())
  pilotRequestId  String
  talentProfileId String
  source          ParticipantSource
  sourcePodId     String?
  createdAt       DateTime        @default(now())

  pilotRequest    PilotRequest    @relation(fields: [pilotRequestId], references: [id], onDelete: Cascade)
  sourcePod       ClientPod?      @relation(fields: [sourcePodId], references: [id], onDelete: SetNull)
  // talentProfile relation is added when the Talent Profile model is finalized

  @@unique([pilotRequestId, talentProfileId])
  @@index([talentProfileId])
}
```

A selected Pod is expanded into one row per current Talent member. `sourcePodId`
is an internal origin reference and is never exposed to Admin as private Pod
content.

## 10. Pilot

```prisma
model Pilot {
  id                   String       @id @default(cuid())
  companyId            String
  pilotRequestId       String       @unique
  status               PilotStatus  @default(PLANNED)
  title                String
  businessGoals        String
  successMeasurements  String
  startDate            DateTime?
  endDate              DateTime?
  cancelledAt          DateTime?
  cancellationReason   String?
  completedAt          DateTime?
  createdAt            DateTime     @default(now())
  updatedAt            DateTime     @updatedAt

  company              Company      @relation(fields: [companyId], references: [id], onDelete: Restrict)
  pilotRequest         PilotRequest @relation(fields: [pilotRequestId], references: [id], onDelete: Restrict)
  participants         PilotParticipant[]
  progressUpdates      PilotProgressUpdate[]

  @@index([companyId, status])
  @@index([endDate])
}
```

The service, not a database enum alone, enforces that only one completed Pilot
exists per Company and that only one planned or active Pilot exists at a time.

## 11. Pilot Participants

```prisma
model PilotParticipant {
  id              String            @id @default(cuid())
  pilotId         String
  talentProfileId String
  source          ParticipantSource
  sourcePodId     String?
  createdAt       DateTime          @default(now())

  pilot           Pilot             @relation(fields: [pilotId], references: [id], onDelete: Cascade)
  sourcePod       ClientPod?        @relation(fields: [sourcePodId], references: [id], onDelete: SetNull)

  @@unique([pilotId, talentProfileId])
  @@index([talentProfileId])
}
```

Participants are copied from the approved request and remain stable even if a
private Pod later changes.

## 12. PilotProgressUpdate

```prisma
model PilotProgressUpdate {
  id          String    @id @default(cuid())
  pilotId     String
  authorId    String
  text        String
  createdAt   DateTime  @default(now())
  removedAt   DateTime?
  removedById  String?

  pilot       Pilot     @relation(fields: [pilotId], references: [id], onDelete: Cascade)
  author      User      @relation("PilotUpdateAuthor", fields: [authorId], references: [id], onDelete: Restrict)

  @@index([pilotId, createdAt])
}
```

The update text, author, and creation date are immutable. Admin removal is a
soft removal for auditability; removed updates are not shown to the Client.

## 13. Private Pods

```prisma
model ClientPod {
  id              String       @id @default(cuid())
  companyId       String
  name            String
  description     String?
  podLeadTalentId String?
  status          PodStatus    @default(ACTIVE)
  createdAt       DateTime     @default(now())
  updatedAt       DateTime     @updatedAt

  company         Company      @relation(fields: [companyId], references: [id], onDelete: Cascade)
  members                 ClientPodMember[]
  requestOrigins          PilotRequestParticipant[]
  pilotOrigins            PilotParticipant[]
  interviewRequestGroups  InterviewRequestGroup[]

  @@index([companyId, status])
}

model ClientPodMember {
  podId           String
  talentProfileId String
  createdAt       DateTime @default(now())

  pod             ClientPod @relation(fields: [podId], references: [id], onDelete: Cascade)

  @@id([podId, talentProfileId])
  @@index([talentProfileId])
}
```

The service must enforce that `podLeadTalentId` is a current member and that all
members are visible, eligible Talent at the time of addition. Admin APIs do not
return Pod records.

## 14. Interview Request Groups and Requests

```prisma
model InterviewRequestGroup {
  id             String    @id @default(cuid())
  companyId      String
  sourcePodId    String?
  requestedById  String
  createdAt      DateTime  @default(now())

  company        Company   @relation(fields: [companyId], references: [id], onDelete: Restrict)
  requests       InterviewRequest[]
  sourcePod      ClientPod? @relation(fields: [sourcePodId], references: [id], onDelete: SetNull)

  @@index([companyId, createdAt])
}

model InterviewRequest {
  id             String                  @id @default(cuid())
  companyId      String
  groupId        String?
  talentProfileId String
  status         InterviewRequestStatus  @default(PENDING)
  origin         InterviewRequestOrigin
  context        String?
  scheduledAt    DateTime?
  timezone       String?
  adminNotes     String?
  rejectionReason String?
  requestedById  String
  createdAt      DateTime                @default(now())
  updatedAt      DateTime                @updatedAt

  company        Company                 @relation(fields: [companyId], references: [id], onDelete: Restrict)
  group          InterviewRequestGroup?  @relation(fields: [groupId], references: [id], onDelete: SetNull)

  @@index([companyId, status])
  @@index([talentProfileId, status])
}
```

The group/source relation supports a Pod batch action while the Admin API
returns only individual Talent requests and client-safe source information.
Internal Admin notes are never returned to the Client.

## 15. Constraints and Enforcement Map

| Rule | Enforcement |
|---|---|
| One primary Client Account per Company | `ClientAccount.companyId @unique` |
| One Client Account per User | `ClientAccount.userId @unique` |
| Client access requires active account | Authorization middleware/service |
| Archived Company blocks Workspace | Authorization middleware/service |
| Only preparation Companies may submit Pilot Requests | Pilot Request service transaction |
| One non-final Pilot process at a time | Company-scoped transaction and status query |
| Only one completed Pilot | Company-scoped transaction and completed-status check |
| Rejected request may resubmit | Request transition service |
| Cancelled Pilot does not consume quota | Completion-only eligibility check |
| Fixed duration | Server sets `durationDays = 14`; client cannot override |
| No duplicate Pod membership | Composite primary key |
| Pod Lead must be a member | Service validation and transaction |
| No duplicate Pilot participant | Composite unique constraint |
| Immutable progress update | No update endpoint; soft-delete only for Admin |
| Private Pods | Client-only repository/API exposure |
| Company-scoped Client data | Session-derived `companyId` on every query |

## 16. Migration Notes

- Add relations to the existing Better Auth `User` model without changing its
  authentication ownership.
- Add an optional Lead-to-Company relation for conversion traceability.
- Add Talent Profile foreign keys when the Talent Profile model is finalized.
- Add indexes for Company status, Pilot status/end date, request status, and
  Client Account status.
- Backfill no Workspace records because no Workspace model is introduced.
