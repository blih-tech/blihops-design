# BlihOps V1 — Product Brief

## 1. Overview

BlihOps is an outsourcing platform that connects global companies with pre-vetted Ethiopian software engineers.

Version 1 validates the outsourcing business by generating qualified leads, running internal sales and talent operations, giving approved clients access to a private talent workspace, and allowing approved talent to maintain permitted parts of an admin-created profile.

The future BlihOps Skills product and public Talent Marketplace are not part of Version 1.

---

## 2. Goals

### Business Goals

- Generate qualified company leads.
- Build and maintain a vetted talent pool.
- Reduce the time required to match clients with engineers.
- Deliver a professional pilot and client experience.
- Keep published website content current without introducing a general-purpose page builder.

### Product Goals

- Centralize lead, company, talent, and content operations.
- Recruit and validate engineers before presenting them to clients.
- Provide approved companies with a secure, private talent workspace.
- Let approved talent maintain accurate professional information.
- Keep permissions and operational control with BlihOps.

---

## 3. Products

### 3.1 BlihOps Web

The public website supports marketing, lead generation, talent applications, careers content, and entry into protected applications.

Key capabilities:

- Marketing pages
- Contact and pilot-request forms
- Book a Call through Calendly
- Join Talent Pool application
- Case Studies and Insights
- Careers roles
- Client and Talent login
- Client Workspace
- Talent Portal

### 3.2 BlihOps Admin

The internal operations platform used by the BlihOps team.

Key capabilities:

- Dashboard
- Lead management
- Company and client-access management
- Talent recruitment
- Talent Profile management
- Interview and pilot oversight
- Bounded website content management
- Email-template and Calendly settings

### 3.3 BlihOps API

The backend powering the public website, Admin Portal, Client Workspace, and Talent Portal.

Primary responsibilities:

- Authentication and role-based authorization
- Invitations and secure tokens
- Lead and company management
- Talent applications and recruitment
- Talent Profiles and Talent Portal access
- Client Workspace and interview requests
- Content management and locale-aware delivery
- Notifications, files, and activity history

---

## 4. User Types

### Visitor

Can:

- Browse public website content.
- Contact BlihOps.
- Request a pilot.
- Book a call.
- Apply to the Talent Pool.
- Browse published Case Studies, Insights, and Careers roles.

### Admin

An internal BlihOps operator who manages every V1 operational workflow and all admin-controlled profile fields.

### Client

The single approved company representative invited by BlihOps.

Can:

- Access the company’s private Client Workspace.
- Browse visible Talent Profiles.
- Search and filter available talent.
- Request interviews through BlihOps.
- Track interview requests and pilot progress.

### Talent

An approved engineer whose Talent Profile has been manually created by an admin.

Can:

- Activate an invitation-only Talent account.
- Access a private Profile Management page.
- Maintain permitted professional information.
- Upload or replace their profile photo and resume.
- Update availability and professional links.

Talent cannot change commercial rates, assessments, verification, client visibility, lifecycle status, or internal notes.

---

## 5. Core Business Lifecycles

### 5.1 Company Lifecycle

```text
Visit Website
      ↓
Contact / Pilot Request / Book Call
      ↓
Lead Created
      ↓
Admin Qualification
      ↓
Company Created
      ↓
Client Workspace Created
      ↓
Client Account Invitation
      ↓
Client Activates Account
      ↓
Client Workspace Access
```

Each company has one Client Workspace and one Client account in Version 1.

### 5.2 Talent Lifecycle

```text
Join Talent Pool
      ↓
Submit Application
      ↓
Application Review
      ↓
Screening and Assessments
      ↓
Admin Approval
      ↓
Profile-Completion Token Sent
      ↓
Talent Submits Final Information
      ↓
Admin Reviews Submission
      ↓
Admin Manually Creates Talent Profile
      ↓
Admin Sends Talent Account Invitation
      ↓
Talent Activates Account
      ↓
Talent Maintains Permitted Profile Information
```

The profile-completion token and Talent account invitation are separate events. Completing the tokenized form does not create an account or Talent Profile automatically.

---

## 6. Client Workspace

The Client Workspace gives the invited company representative a focused, private way to evaluate available engineers and follow the pilot process.

V1 capabilities:

- Dashboard
- Talent Directory
- Talent Profile details
- Search, filters, sorting, and pagination
- Interview Requests
- Pilot Status

Clients cannot edit Talent Profiles, contact talent directly, create teams, or invite additional client users.

---

## 7. Talent Portal

The Talent Portal is a single authenticated Profile Management experience. It does not contain a dashboard.

Talent may update:

- Profile photo
- Professional headline
- Short bio
- Primary role
- Tech stack and secondary skills
- Years of experience
- Portfolio, GitHub, and LinkedIn links
- Resume
- Availability
- Earliest start date
- Preferred engagement

Permitted changes publish immediately. Admin-only fields remain protected.

---

## 8. Talent Pool

The Talent Pool is the admin-managed database of approved engineers.

Profiles may include:

- Profile photo and identity
- Headline, bio, role, and seniority
- Tech stack, skills, and experience
- Portfolio, GitHub, LinkedIn, and resume
- English level and verification
- Availability and engagement preference
- Hourly and monthly rates
- Assessment summary and internal notes
- Visibility and lifecycle status

Only visible, active Talent Profiles appear in the Client Workspace.

---

## 9. Managed Website Content

V1 provides structured content management for selected website content only:

- Home: trusted logos, testimonials, and one primary testimonial.
- Services: one hero video with a cover image.
- Case Studies: English and German content.
- Insights: English and German content.
- Careers: English-only roles.
- Pilot pages: English and German FAQs.

Case Studies, Insights, and Pilot FAQs require complete English and German content before publication. BlihOps does not provide unrestricted page editing in V1.

---

## 10. Product Shape

```text
BlihOps Web
├── Public Website
├── Client Workspace
└── Talent Portal

BlihOps Admin
├── Operations
└── Managed Content

All applications
└── BlihOps API
```

---

## 11. Out of Scope for Version 1

- BlihOps Skills
- Public Talent Marketplace
- Public account registration
- Multiple Client accounts per company
- Client team management
- Talent-created profiles before admin approval
- Talent access to commercial, assessment, verification, visibility, or internal fields
- Direct client-to-talent messaging
- Contracts, payments, and invoicing
- AI talent matching
- Advanced analytics and reporting
- Multi-factor authentication
- General-purpose website page building

