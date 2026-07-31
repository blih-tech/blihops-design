# Information Architecture

## 1. Overview

This document defines how BlihOps V1 information is organized across the public website, Admin Portal, Client Workspace, and Talent Portal.

It derives from the canonical V1 PRD and provides the structural foundation for navigation, user flows, screen inventory, and UI design.

---

## 2. Architecture Principles

- Separate public, admin, client, and talent experiences.
- Keep Admin operations resource-oriented.
- Keep Client and Talent experiences focused on their primary tasks.
- Use shallow, predictable navigation.
- Give every major resource a stable, permission-aware URL.
- Keep managed website content structured rather than page-based.
- Do not expose internal, commercial, or assessment data outside Admin.

---

## 3. Product Hierarchy

```text
BlihOps
├── Public Website
│   ├── Marketing Pages
│   ├── Lead Generation
│   ├── Talent Applications
│   ├── Published Content
│   └── Protected Application Entry
│
├── Admin Portal
│   ├── Dashboard
│   ├── Leads
│   ├── Companies
│   ├── Talent Applications
│   ├── Talent Profiles
│   ├── Interview and Pilot Oversight
│   ├── Managed Content
│   └── Settings
│
├── Client Workspace
│   ├── Dashboard
│   ├── Talent Directory
│   ├── Interview Requests
│   └── Pilot Status
│
└── Talent Portal
    └── Profile Management
```

---

## 4. Primary Application Structures

### 4.1 Public Website

```text
Public Website
├── Home
├── Services
├── How We Work
├── About
├── Case Studies
│   └── Case Study Detail
├── Insights
│   └── Insight Detail
├── Careers
├── Pilot
├── Contact
├── Join Talent Pool
├── Login
├── Privacy
└── Terms
```

Public actions create Leads or Talent Applications but do not expose those resources back to the Visitor.

### 4.2 Admin Portal

```text
Admin Portal
├── Dashboard
├── Leads
│   ├── Active Leads
│   ├── Archived Leads
│   └── Lead Detail
├── Companies
│   ├── Active Companies
│   ├── Archived Companies
│   └── Company Detail
├── Talent Applications
│   ├── Recruitment Pipeline
│   ├── Approved
│   ├── Rejected
│   ├── Archived
│   └── Application Detail
├── Talent Profiles
│   ├── Visible
│   ├── Hidden
│   ├── Archived
│   └── Talent Profile Detail
├── Interview Requests
├── Managed Content
│   ├── Trusted Logos
│   ├── Testimonials
│   ├── Services Hero Media
│   ├── Case Studies
│   ├── Insights
│   ├── Careers Roles
│   └── Pilot FAQs
└── Settings
    ├── Email Templates
    └── Calendly
```

### 4.3 Client Workspace

```text
Client Workspace
├── Dashboard
├── Talent Directory
│   └── Talent Profile Detail
├── Interview Requests
│   └── Interview Request Detail
└── Pilot Status
```

Each Company has one Client Workspace and one Client account in V1.

### 4.4 Talent Portal

```text
Talent Portal
└── Profile Management
    ├── Professional Information
    ├── Skills and Experience
    ├── Links and Resume
    └── Availability
```

Profile Management is a single application destination. The listed groups are sections of the page, not separate primary destinations.

---

## 5. Resource Hierarchy

```text
Platform
├── Leads
├── Companies
│   └── Client Workspace
│       └── Interview Requests
├── Talent Applications
│   ├── Profile-Completion Submission
│   └── Talent Profile
│       └── Talent Account
├── Managed Content
│   ├── Trusted Logos
│   ├── Testimonials
│   ├── Services Hero Media
│   ├── Case Studies
│   ├── Insights
│   ├── Careers Roles
│   └── Pilot FAQs
└── Settings
```

---

## 6. Resource Relationships

```text
Lead
  └── may convert to → Company
                         └── owns → Client Workspace
                                      └── contains → Interview Requests

Talent Application
  ├── may receive → Profile-Completion Submission
  └── may produce → Talent Profile
                       └── may enable → Talent Account

Interview Request
  ├── belongs to → Company / Client Workspace
  └── references → Talent Profile
```

Relationship rules:

- A Lead may convert into one Company.
- Every Company has one Client Workspace.
- Every Company has at most one Client account in V1.
- Every Talent Application may produce one Talent Profile.
- A profile-completion submission belongs to one approved Talent Application.
- A Talent account can exist only after its Talent Profile exists.
- Every Interview Request belongs to one Company and references one Talent Profile.
- There is no Team resource in V1.

---

## 7. Information Ownership and Access

| Resource | Owner or Parent | Admin | Client | Talent | Visitor |
|----------|-----------------|-------|--------|--------|---------|
| Lead | Platform | Full | None | None | Create through forms |
| Company | Platform | Full | Own context, read-only | None | None |
| Client Workspace | Company | Full | Own workspace | None | None |
| Interview Request | Client Workspace | Full | Own company, create/read | None | None |
| Talent Application | Platform | Full | None | Token-scoped completion only | Create |
| Talent Profile | Talent Application | Full | Visible fields only | Own permitted fields | None |
| Managed Content | Platform | Full | Public output only | Public output only | Published output |
| Settings | Platform | Full | None | None | None |

---

## 8. Managed Content Structure

### Global, Non-Localized Content

- Trusted Logos
- Testimonials
- Primary Testimonial selection
- Services Hero Media
- Careers Roles in English

### Bilingual Content

- Case Studies
- Insights
- Pilot FAQs

Bilingual content uses one record with English and German content groups. Both locale groups must be complete before the shared record can publish or activate.

---

## 9. URL and Deep-Link Model

Every major resource has a stable URL where permissions allow:

- Lead
- Company
- Talent Application
- Talent Profile
- Interview Request
- Case Study
- Insight
- Careers Role

Tokenized URLs are used only for:

- Client account invitation
- Talent profile-completion request
- Talent account invitation
- Password reset

Tokenized links validate before showing protected form content.

---

## 10. Explicit V1 Boundaries

- The Talent Portal has no dashboard.
- The Client Workspace has no Teams section or Team resource.
- Companies cannot manage multiple Client users.
- Talent cannot create a profile before Admin approval.
- Managed Content does not expose arbitrary page editing.

