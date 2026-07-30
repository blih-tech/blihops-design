# Navigation

## Overview

This document defines how users navigate through BlihOps. It describes the navigation structure for each application, global navigation patterns, and navigation principles that ensure a fast, predictable, and consistent user experience.

---

# Navigation Principles

Navigation should be:

- Simple and predictable
- Consistent across each application
- Minimize the number of clicks
- Keep users oriented within their current application
- Prioritize frequently used actions
- Clearly separate operational and client experiences

---

# Navigation Hierarchy

```text
Website
├── Home
├── Services
├── About
├── Contact
├── Request Pilot
├── Book a Call
├── Join Talent Pool
└── Client Login

Admin Portal
├── Dashboard
├── Leads
├── Companies
├── Talent Applications
├── Talent Profiles
├── CMS
└── Settings

Client Workspace
├── Dashboard
├── Talent Directory
├── Interview Requests
├── Teams
└── Pilot Status

Talent Portal
├── Dashboard
└── Profile Management
```

---

# Primary Navigation

Each application provides its own primary navigation.

## Website

- Home
- Services
- About
- Contact
- Request Pilot
- Book a Call
- Join Talent Pool
- Client Login

---

## Admin Portal

- Dashboard
- Leads
- Companies
- Talent Applications
- Talent Profiles
- CMS
- Settings

---

## Client Workspace

- Dashboard
- Talent Directory
- Interview Requests
- Teams
- Pilot Status

---

## Talent Portal

- Dashboard
- Profile Management

---

# Secondary Navigation

Some sections provide local navigation for related views.

### Leads

- Contact Requests
- Pilot Requests
- Calendly Bookings
- Archived

### Companies

- Active
- Archived

### Talent Applications

- Active Pipeline
- Approved
- Rejected

### Talent Profiles

- Visible
- Hidden
- Archived

### Settings

- Email Templates
- Calendly
- General

---

# Global Navigation Elements

Available within authenticated applications where applicable:

- User Menu
- Profile
- Logout

---

# Navigation Patterns

Users should be able to:

- Navigate between sections without losing context.
- Return to the previous page using browser navigation.
- Open resources directly from contextual links.
- Move between related resources with minimal navigation.

---

# Deep Linking

Every major resource should have a unique URL.

Examples:

- Lead
- Company
- Talent Application
- Talent Profile
- Team
- Interview Request

Users should be able to bookmark or share links to individual resources where permissions allow.

---

# Navigation Behavior

- The active section is clearly highlighted.
- Navigation remains consistent within each application.
- Navigation state persists during normal browsing.
- Unauthorized navigation redirects users to the appropriate login page or access-denied screen.
- Users cannot navigate between applications unless they have permission.

---

# Future Enhancements

- Global Search
- Command Palette
- Recently Viewed
- Favorites
- Breadcrumb Navigation