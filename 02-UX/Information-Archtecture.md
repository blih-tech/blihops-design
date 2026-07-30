# Information Architecture

## Overview

This document defines how information is organized within BlihOps. It establishes the hierarchy of the platform, the relationships between applications and resources, and the logical grouping of functionality.

The Information Architecture serves as the foundation for navigation, screen design, user flows, and implementation.

---

# Architecture Principles

The information architecture follows these principles:

- Application-first organization
- Clear separation of user experiences
- Predictable hierarchy
- Minimal navigation depth
- Consistent resource relationships
- Scalable for future products

---

# Product Hierarchy

```text
BlihOps
├── Website
│   ├── Marketing Pages
│   ├── Lead Generation
│   ├── Talent Applications
│   └── Client Workspace Entry
│
├── Admin Portal
│   ├── Dashboard
│   ├── Leads
│   ├── Companies
│   ├── Talent Applications
│   ├── Talent Profiles
│   ├── CMS
│   └── Settings
│
├── Client Workspace
│   ├── Dashboard
│   ├── Talent Directory
│   ├── Interview Requests
│   ├── Teams
│   └── Pilot Status
│
└── Talent Portal
    ├── Dashboard
    └── Profile Management
```

---

# Resource Hierarchy

```text
Platform
│
├── Leads
├── Companies
│   └── Client Workspace
│       ├── Teams
│       └── Interview Requests
│
├── Talent Applications
│   └── Talent Profiles
│
├── Users
│   ├── Admin
│   ├── Client
│   └── Talent
│
├── CMS
└── Settings
```

---

# Primary Sections

## Website

Public-facing application responsible for marketing, lead generation, talent recruitment, and client entry.

Contains:

- Marketing Pages
- Contact
- Pilot Request
- Book a Call
- Join Talent Pool
- Client Login

---

## Admin Portal

Internal platform used to operate the business.

Contains:

- Dashboard
- Leads
- Companies
- Talent Applications
- Talent Profiles
- CMS
- Settings

---

## Client Workspace

Private workspace for approved clients.

Contains:

- Dashboard
- Talent Directory
- Talent Profiles
- Interview Requests
- Teams
- Pilot Status

---

## Talent Portal

Self-service portal for approved engineers.

Contains:

- Dashboard
- Profile Management

---

# Resource Relationships

```text
Lead
    ↓
Company
    ↓
Client Workspace
    ├── Teams
    └── Interview Requests

Talent Application
        ↓
Talent Profile
        ↓
Talent Portal
```

### Relationship Rules

- Every Lead may become one Company.
- Every Company has one Client Workspace.
- Every Talent Application may become one Talent Profile.
- Every Talent Profile belongs to one Talent.
- Every Team belongs to one Client Workspace.
- Every Interview Request belongs to one Company and one Talent Profile.

---

# Information Ownership

| Resource | Parent |
|-----------|--------|
| Lead | Platform |
| Company | Platform |
| Client Workspace | Company |
| Team | Client Workspace |
| Interview Request | Client Workspace |
| Talent Application | Platform |
| Talent Profile | Talent Application |
| Talent Portal | Talent |
| CMS | Platform |
| Settings | Platform |

---

# Future Expansion

The architecture is designed to support future products without restructuring the platform.

Potential future sections include:

- Skills Platform
- Talent Marketplace
- Messaging
- Payments
- Contracts
- Analytics
- AI Matching
- Integrations