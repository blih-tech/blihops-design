# BlihOps V1 Information Architecture

## 1. Purpose

This document defines the V1 product hierarchy, navigation groups, major content
relationships, and role-based access boundaries. Detailed interaction steps,
screen states, and edge cases belong in the User Flows and Screen Inventory.

## 2. Product Structure

```text
BlihOps
├── BlihOps Web
│   ├── Public Website
│   ├── Protected Client Workspace
│   ├── Protected Talent Portal
│   └── Shared Authentication and Invitation Routes
├── BlihOps Admin
└── BlihOps API
```

The Client Workspace and Talent Portal are protected areas within BlihOps Web;
they are not separate platforms. BlihOps Admin is the internal operations
application. The API supports all interfaces but does not have a user-facing
navigation structure.

## 3. BlihOps Web Architecture

### 3.1 Public Website

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
│   └── Career Role Detail
├── Pilot
│   ├── Pilot Request
│   └── Book a Discovery Call
├── Contact
├── Join Talent Pool
├── Login
├── Privacy
└── Terms
```

The public website supports English and German content where available. Locale
selection changes the content variant without creating a separate site
hierarchy. Client and Talent users share one login screen. After successful
authentication, the system checks the user's role and redirects them to the
Client Workspace or Talent Profile Management respectively.

### 3.2 Protected Client Workspace

```text
Client Workspace
├── Dashboard
├── Talent Directory
│   └── Talent Profile
├── Talent Pods
│   └── Pod Detail
├── Interview Requests
├── Pilot Status
└── Account Actions
    ├── Password Reset
    └── Logout
```

The Workspace belongs to one Company and is accessible by its single active
Client account. Talent Profiles are reached through the Directory, Pods, and
related Interview Requests. Pods are private planning groups and do not create
assignments or direct contact with Talent.

### 3.3 Protected Talent Portal

```text
Talent Portal
├── Profile Management
└── Account Actions
    ├── Password Reset
    └── Logout
```

The Talent Portal is a focused protected route with no dashboard. Approved
Talent land directly on their own Profile Management page and can edit only the
permitted fields of the Admin-created profile.

### 3.4 Shared and Temporary Routes

The following routes support entry, recovery, and handoffs but do not appear in
primary navigation:

- Shared account invitation activation (`/accept-invitation`)
- Profile completion form (`/complete-profile`) — token-protected, unauthenticated
- Password-reset request and new-password form
- Invalid, expired, or consumed link
- Access denied
- Submission confirmation

Client and Talent account invitations use the same `/accept-invitation` screen.
The invitation token identifies the account type, and successful activation
redirects the user to the Client Workspace or Talent Profile Management based on
that role. The profile completion form uses a separate, single-use 7-day token
sent to the candidate after application approval; it precedes the account
invitation and does not log the candidate into the platform.

## 4. BlihOps Admin Architecture

```text
BlihOps Admin
├── Dashboard
├── Leads
│   └── Lead Detail
├── Companies
│   └── Company Detail
│       ├── Client and Workspace Access
│       ├── Pilot Status and Milestones
│       ├── Interview Requests
│       ├── Notes
│       └── Activity
├── Talent Applications
│   └── Application Detail
├── Talent Profiles
│   └── Talent Profile Detail
├── Content
│   ├── Trusted Logos
│   ├── Testimonials
│   ├── Services Hero Media
│   ├── Case Studies
│   ├── Insights
│   ├── Careers Roles
│   └── Pilot FAQs
├── Settings
│   ├── Email Templates
│   └── Cal.com
└── Account Actions
    └── Logout
```

The Admin Dashboard provides operational summaries and entry points. Notes and
activity remain contextual to their related resources rather than becoming
primary navigation sections. Company Pilot management remains within Company
Detail. Interview Requests are also viewed and managed within the related
Company Detail rather than through a separate page.

## 5. Access and Visibility Rules

| Area | Access | Information boundary |
|---|---|---|
| Public Website | Anyone | Published or active public content only |
| Client Workspace | Invited Client | Their Company, Pods, requests, Pilot, and visible Talent Profiles |
| Talent Portal | Invited Talent | Their own profile and permitted editable fields |
| BlihOps Admin | Provisioned Admin | All authorized V1 operational and managed-content resources |

- There is no public account registration.
- BlihOps Web uses one login screen for Client and Talent accounts.
- Successful authentication redirects each user to the destination permitted by
  their role.
- Clients cannot access another Company's Workspace or internal Talent data.
- Talent cannot access another profile or edit Admin-managed fields.
- Hidden or archived Talent Profiles are unavailable to Clients.
- Archiving a Company disables access to its Client Workspace without deleting
  its history.

## 6. Navigation Principles

- Public navigation prioritizes understanding the service, evaluating trust,
  contacting BlihOps, requesting a Pilot, and joining the Talent Pool.
- Client navigation prioritizes discovering Talent, organizing Pods, requesting
  interviews, and tracking the Pilot.
- Talent navigation remains limited to Profile Management and account actions.
- Admin navigation follows the operational domains of sales, Companies, Talent,
  delivery oversight, managed content, and settings.
- Notifications, confirmations, and activity records appear in context unless a
  later requirement establishes a dedicated notification center.
- Protected or unavailable destinations provide an appropriate login,
  access-denied, expired-link, or unavailable-resource state.
