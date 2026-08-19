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
│   ├── Protected Client Workspace experience
│   ├── Protected Talent Portal
│   └── Shared Authentication and Invitation Routes
├── BlihOps Admin
└── BlihOps API
```

The Client Workspace is a protected Web experience scoped by the authenticated
Client Account's Company ID. It is not a separate Workspace product or
Database model in V1. BlihOps Admin is the internal operations application.

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
│   ├── Public Pilot Request
│   └── Book a Discovery Call
├── Contact
├── Join Talent Pool
├── Login
├── Privacy
└── Terms
```

The public Pilot form creates a Lead. It is separate from the authenticated
Client Pilot Request inside the Client Workspace.

### 3.2 Protected Client Workspace

```text
Client Workspace
├── Work
│   ├── Pilot Preparation
│   ├── Pilot Request Review
│   ├── Planned Pilot
│   ├── Active Pilot
│   ├── Completed Pilot History
│   └── Active Partnership
├── Talents
│   └── Talent Detail
├── Pods
│   └── Pod Detail
└── Interview Requests
```

The client enters `/client-workspace`. The API resolves the Company from the
Client Account in the authenticated session. No Workspace ID is required in the
client URL.

The Workspace belongs logically to one Company and is accessible only when the
Client Account is active and the Company is not archived.

Pods are private Client planning groups. Admin sees only the individual
Interview Requests and Pilot participants resulting from explicit Client
actions, not the private Pod structure.

### 3.3 Protected Talent Portal

```text
Talent Portal
└── Profile Management
```

Approved Talent land directly on their own Profile Management page. There is no
Talent Dashboard in V1.

### 3.4 Shared and Temporary Routes

- Shared account invitation activation (`/accept-invitation`)
- Profile completion form (`/complete-profile`), token-protected and unauthenticated
- Password reset request and new-password form
- Invalid, expired, or consumed link state
- Access denied state
- Archived Company or deactivated Client Account state
- Submission confirmation

Successful activation redirects by role:

```text
CLIENT → /client-workspace
TALENT → Talent Profile Management
```

## 4. BlihOps Admin Architecture

```text
BlihOps Admin
├── Dashboard
├── Leads
│   └── Lead Detail
├── Companies
│   └── Company Detail
│       ├── Company Profile
│       ├── Client Account and Invitation
│       ├── Workspace Access State
│       ├── Interview Requests
│       ├── Pilot Requests and Pilot History
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

Interview Requests and Pilot management remain within Company Detail. There is
no separate Admin Pod management screen because Pods are Client-private.

## 5. Access and Visibility Rules

| Area | Access | Information boundary |
|---|---|---|
| Public Website | Anyone | Published or active public content only |
| Client Workspace | Active invited Client | Their Company, private Pods, requests, Pilot data, curated activity, and visible Talent |
| Talent Portal | Active invited Talent | Their own profile and permitted editable fields |
| BlihOps Admin | Provisioned Admin | Authorized operational and managed-content resources |

- There is no public account registration.
- Client access requires `ClientAccount.status = ACTIVE`.
- Company `ARCHIVED` blocks Client Workspace access.
- Deactivated Client Accounts cannot log in or use existing sessions.
- Clients cannot access another Company's data.
- Clients cannot see internal Admin notes, audit history, or commercial Talent data.
- Admin cannot see private Pod content.
- Hidden Profiles and deactivated Talent Accounts are unavailable to Clients.

## 6. Navigation Principles

- Public navigation prioritizes understanding the service, contacting Blihops,
  requesting a public Pilot, and joining the Talent Pool.
- Client navigation prioritizes discovering Talent, organizing private Pods,
  requesting interviews, submitting the one free Pilot Request, and monitoring
  Pilot history.
- Admin navigation follows operational domains: Leads, Companies, Talent,
  delivery oversight, content, and settings.
- Work is the first Client page and supports Pilot preparation, Pilot monitoring,
  completed history, and simple Active partnership statistics.
- V1 has no contracts, billing, tasks, daily work management, milestones, or
  notification center.
- Transactional email is limited to invitations, profile completion requests,
  and password resets.
