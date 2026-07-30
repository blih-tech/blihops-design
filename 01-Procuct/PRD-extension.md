# BlihOps V2 PRD Extension

# 4. User Roles

## Talent

Approved software engineer invited by BlihOps.

### Can:

- Access the Talent Portal.
- View and update their professional profile.
- Update portfolio links.
- Update resume.
- Update skills and experience.
- Update availability.
- Update professional information permitted by BlihOps.

### Business Rules

- Only approved talents receive Talent Portal access.
- Talents can only access their own profile.
- Talents cannot edit internal assessment data.
- Talents cannot edit approval status or visibility settings.
- All profile changes are immediately reflected in the Talent Profile unless otherwise configured by the admin.

---

# 7. Talent Portal

## Purpose

Provide approved talents with a secure self-service portal to maintain and improve their professional profile.

---

## Dashboard

### Displays

- Profile Completion
- Profile Status
- Availability
- Recent Updates

---

## Profile Management

Talents can update:

- Profile Photo
- Professional Headline
- Short Bio
- Primary Role
- Tech Stack
- Secondary Skills
- Years of Experience
- Portfolio
- GitHub
- LinkedIn
- Resume
- Availability
- Earliest Start Date
- Preferred Engagement

---

## Functional Requirements

Talent can:

- View Profile
- Edit Profile
- Upload Resume
- Upload Profile Photo
- Update Portfolio Links
- Update Availability

---

## Business Rules

- Talents can only modify their own profile.
- Internal assessment information is read-only.
- Profile ownership belongs to the talent while verification remains under BlihOps.
- Authentication is required for all Talent Portal access.

---

# 8. Client Workspace

## Team Management

### Purpose

Allow clients to organize hired engineers into project teams.

---

## Team List

### Displays

- Team Name
- Number of Members
- Pod Lead
- Created Date

---

## Team Details

### Displays

- Team Name
- Pod Lead
- Team Members

---

## Functional Requirements

Client can:

- Create Team
- Rename Team
- Delete Team
- Add Hired Talent
- Remove Hired Talent
- Move Talent Between Teams
- Assign Pod Lead
- Remove Pod Lead

---

## Business Rules

- Teams exist only within the client's workspace.
- A client can create multiple teams.
- Team names are unique within a workspace.
- Pod Lead assignment is optional.
- Only hired talents can be assigned to teams.
- Team management does not affect the hiring workflow.
- Hiring and interview coordination continue to be managed manually by BlihOps.

---

# 12. Business Rules

## Talent Portal

- Only approved talents receive portal access.
- Every talent has one Talent Portal account.
- Talents may update only editable profile fields.
- Internal verification fields remain editable only by admins.

---

## Team Management

- Teams belong to a single client workspace.
- Teams are used to organize hired engineers.
- Teams do not represent recruitment status.
- Pod Lead assignment is optional.
- Engineers may be reassigned between teams by the client.

---

# 13. Assumptions & Future Considerations

Current assumptions for Version 2:

- One Talent account per approved Talent Profile.
- One Client Workspace per company.
- Clients organize hired engineers using Teams.
- Hiring and recruitment continue to be managed manually by BlihOps.
- Team management is organizational only and does not replace recruitment workflows.