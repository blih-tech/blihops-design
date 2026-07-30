# BlihOps V1 – Product Requirements Document (PRD)

# 1. Product Overview

## Purpose

BlihOps is an outsourcing platform that connects global companies with pre-vetted Ethiopian software engineers.

Version 1 focuses on validating the business by:

- Generating qualified company leads.
- Building and maintaining a vetted talent pool.
- Providing approved clients with a private workspace to explore available talent.
- Managing all operations through an internal admin portal.

Version 1 intentionally excludes the future Skills platform and Talent Marketplace.

---

# 2. Product Goals

## Business Goals

- Generate qualified leads.
- Build a high-quality talent pool.
- Reduce time-to-match for clients.
- Deliver an excellent pilot experience.

## Product Goals

- Centralize operations.
- Recruit and validate talent.
- Provide a secure client workspace.
- Keep the platform simple and scalable.

---

# 3. Products

## 3.1 BlihOps Web

### Purpose

Public-facing website for marketing, lead generation, talent applications, and client access.

### Features

- Marketing Pages
- Contact Form
- Pilot Request
- Book a Call (Calendly)
- Join Talent Pool
- Client Workspace (Protected)

---

## 3.2 BlihOps Admin

### Purpose

Internal operations platform.

### Features

- Dashboard
- Leads
- Companies
- Talent Applications
- Talent Profiles
- CMS
- Settings

---

## 3.3 BlihOps API

### Purpose

Backend powering both applications.

### Responsibilities

- Authentication
- Lead Management
- Company Management
- Talent Management
- Client Workspace
- CMS
- Notifications

---

# 4. User Roles

## Visitor

Can:

- Browse website
- Contact BlihOps
- Request Pilot
- Book Call
- Apply to Talent Pool

---

## Client

Approved company representative.

Can:

- Access Client Workspace
- Browse Talent Profiles
- View Talent Details
- Shortlist Talent (Future)
- Request Interviews
- View Pilot Status

---

## Admin

Internal BlihOps team.

Can manage every aspect of the platform.

---

# 5. Authentication & Authorization

## Purpose

Provide secure access to protected applications.

## Authentication Architecture

- Single authentication service.
- Single User entity.
- Role-based authorization.

### Roles

- ADMIN
- CLIENT

### Protected Applications

| Application | URL | Access |
|-------------|-----|--------|
| Admin Portal | admin.blihops.com | Admin |
| Client Workspace | blihops.com/workspace | Client |

---

## Authentication Flows

### Admin

```text
Login
  ↓
Admin Dashboard
```

### Client

```text
Invitation Email
      ↓
Accept Invitation
      ↓
Set Password
      ↓
Login
      ↓
Client Workspace
```

### Password Reset

```text
Forgot Password
      ↓
Reset Link
      ↓
New Password
      ↓
Login
```

---

## Functional Requirements

### Admin

- Login
- Logout
- Forgot Password
- Reset Password

### Client

- Accept Invitation
- Login
- Logout
- Forgot Password
- Reset Password

---

## Business Rules

- No public registration.
- Clients are created only by admins.
- Invitations are single-use.
- Invitations expire.
- Non-admin users cannot access the Admin Portal.
- Clients can only access their own Workspace.
- Admin Portal is not indexed by search engines.

---

# 6. BlihOps Web

## Marketing Pages

- Home
- Services
- About
- Contact

---

## Contact Form

### Purpose

General inquiries.

### Hidden Field

- Source Page

### Creates

- Lead (Type: Contact)

---

## Pilot Request

### Purpose

Qualified business leads.

### Hidden Field

- Source Page

### Creates

- Lead (Type: Pilot)

### After successful submission

```text
Success Page
      ↓
Book Discovery Call (Calendly)
```

---

## Book a Call

Calendly integration.

Calendly webhook creates:

- Lead (Type: Calendly)

---

## Join Talent Pool

### Purpose

Collect talent applications.

### Flow

```text
Join Talent Pool
      ↓
Submit Application
      ↓
Application Created
      ↓
Confirmation Page
```

### Creates

- Talent Application

---

# 7. Client Workspace

## Purpose

Private workspace for approved companies.

---

## Dashboard

### Displays

- Welcome
- Pilot Status
- Available Talent
- Recent Activity
- Upcoming Interviews

---

## Talent Directory

### Features

- Search
- Filters
- Sorting
- Pagination

---

### Talent Card

- Photo
- Name
- Headline
- Experience
- Tech Stack
- English Level
- Availability
- Hourly Rate

---

## Talent Profile

### Displays

- Profile Photo
- Professional Information
- Experience
- Tech Stack
- Portfolio
- Resume
- Availability
- Rates

### Actions

- Request Interview

---

## Interview Requests

Client can

- View Requests
- Track Status

### Statuses

- Pending
- Scheduled
- Completed
- Cancelled

---

## Pilot Status

### Displays

- Current Pilot
- Timeline
- Status

---

## Business Rules

- Clients only see Visible Talent Profiles.
- Clients cannot edit Talent Profiles.
- Clients cannot contact talents directly.
- All interview requests go through BlihOps.

# 8. Admin Portal

## Dashboard

### Widgets

- Leads
- Pilot Requests
- Booked Calls
- Talent Applications
- Talent Profiles
- Companies
- Recent Activity

---

## Leads

### Purpose

Manage incoming business leads.

### Sections

- Contact Requests
- Pilot Requests
- Calendly Bookings

### Statuses

- New
- Contacted
- In Discussion
- Closed
- Archived

### Actions

- View
- Edit
- Change Status
- Archive

---

## Companies

### Purpose

Manage approved clients.

Creating a Company automatically:

- Creates Client Workspace

Admin can:

- Send Invitation
- Resend Invitation
- Archive Client

### Statuses

- Invitation Pending
- On Pilot
- Active
- Archived

---

### Company Details

#### Tabs

##### Workspace

Displays

- Company Information
- Workspace Status
- Invitation Status
- Pilot Status
- Workspace URL

### Actions

- Send / Resend Invitation

---

### Activity

Timeline including

- Client Created
- Invitation Sent
- Invitation Accepted
- Viewed Talent Profile
- Requested Interview
- Pilot Started
- Pilot Completed

---

### Notes

Internal notes.

---

## Talent Applications

### Purpose

Recruitment pipeline.

### Statuses

- Applied
- Review
- Screening Call
- Technical Test
- Interview
- Approved
- Profile Completion Requested
- Profile Submitted
- Rejected

### Actions

- Review
- Next
- Reject
- Request Profile Completion
- Create Talent Profile

---

### Expanded View

Displays

- Original Application
- Recruitment History
- Profile Completion Submission

---

## Profile Completion

After approval

Admin requests profile completion.

Talent receives

- Secure token-based email.

Completes

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

## Talent Profiles

### Purpose

Approved talent database.

---

### List

- Photo
- Name
- Headline
- Role
- Seniority
- Availability
- Hourly Rate
- Visibility
- Status

### Actions

- Create
- View
- Edit
- Hide / Show
- Archive

---

### Talent Profile

#### Basic

- Photo
- Name
- Headline
- Bio

#### Professional

- Primary Role
- Seniority
- Years of Experience
- Primary Tech Stack
- Secondary Skills
- Portfolio
- Resume
- GitHub
- LinkedIn

#### Availability

- Status
- Earliest Start Date
- Preferred Engagement

#### Commercial

- Hourly Rate
- Monthly Rate

#### Verification

- English Level
- BlihOps Verified

#### Visibility

- Visible to Clients

#### Internal

- Assessment Summary
- Internal Notes
- Linked Talent Application

---

### Business Rules

- One Talent Profile per approved application.
- Talent Profiles are created from Talent Applications.
- Hidden Talent Profiles are not visible to clients.

---

## CMS

Manage website content.

Future editable sections include:

- Homepage
- Services
- Testimonials
- FAQs

---

## Settings

### Email Templates

- Client Invitation
- Password Reset
- Profile Completion Request

### Calendly

- Webhook Configuration
- Event Mapping
- Connection Status
- Last Webhook Received

---

# 9. Notifications

## Internal

- New Contact Request
- New Pilot Request
- New Talent Application
- New Calendly Booking
- Profile Completion Submitted

---

## Client

- Workspace Invitation
- Password Reset

---

## Talent

- Profile Completion Request

---

# 10. File Management

## Talent Application

- Resume (Required)

---

## Talent Profile

- Profile Photo
- Resume

---

## Rules

- Resume must be PDF.
- Photos support JPG, PNG, WebP.
- Replacing a file overwrites the previous version.
- Files remain available for historical purposes after profile archival.

---

# 11. Audit & Activity

## Lead

- Created
- Updated
- Archived

---

## Company

- Created
- Workspace Created
- Invitation Sent
- Invitation Accepted
- Archived

---

## Talent Application

- Submitted
- Status Changed
- Profile Completion Requested
- Profile Submitted

---

## Talent Profile

- Created
- Updated
- Visibility Changed
- Archived

---

## Client Workspace

- Talent Viewed
- Interview Requested

---

Each activity records:

- Timestamp
- User / System
- Action
- Target Entity

---

# 12. Business Rules

## Authentication

- Single authentication system.
- Role-based authorization.
- No public registration.

---

## Leads

- Every Contact, Pilot Request, and Calendly booking creates a Lead.
- Leads never automatically become Clients.
- Client creation is manual.
- Leads can be archived.

---

## Companies

- Every Client has one Workspace.
- Workspace is created when the Client is created.
- Invitations are sent manually.
- Archived Clients lose Workspace access.

---

## Talent Applications

- Every submission creates one Talent Application.
- Applications can be rejected at any stage.
- Only approved applications can request profile completion.

---

## Talent Profiles

- One Talent Profile per approved application.
- Clients only see Visible profiles.
- Clients never contact talents directly.

---

## Client Workspace

- Accessible only by invited Clients.
- Clients only access their own Workspace.
- Interview requests are managed through BlihOps.

---

# 13. Assumptions & Future Considerations

Current assumptions for Version 1:

- One seeded admin account.
- One client user per company.
- One workspace per client.
- Recruitment communication handled manually.
- Interview scheduling outside the platform.
- Calendly used for scheduling.
- Talent Profiles managed only by admins.

---

# 14. Out of Scope (Version 1)

## Products

- BlihOps Skills
- BlihOps Talent Marketplace

---

## Authentication

- Public Registration
- Multi-Factor Authentication
- Internal User Management
- Multiple Client Users

---

## Recruitment

- Self-managed Talent Profiles
- AI Matching
- Public Talent Marketplace

---

## Client Features

- Messaging
- Contracts
- Payments
- Invoicing
- Team Management
- Analytics

---

## Admin Features

- Advanced Reporting
- Role-based Admin Permissions
- Workflow Automation

---

# End of PRD