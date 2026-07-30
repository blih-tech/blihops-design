# Navigation

## 1. Overview

This document defines V1 navigation for the public website, Admin Portal, Client Workspace, and Talent Portal.

Navigation follows the canonical PRD and Information Architecture.

---

## 2. Navigation Principles

- Keep each application’s navigation distinct.
- Prioritize frequent tasks.
- Maintain a stable active state.
- Preserve useful list filters when returning from details.
- Support browser back/forward behavior.
- Deep-link major resources when authorization permits.
- Use contextual actions for resource operations instead of adding unnecessary primary destinations.

---

## 3. Public Website Navigation

Primary destinations:

- Home
- Services
- How We Work
- About
- Case Studies
- Insights
- Careers
- Pilot
- Contact

Primary actions:

- Request a Pilot
- Book a Call
- Join Talent Pool
- Log In

Footer destinations:

- Privacy
- Terms
- Contact
- Relevant marketing destinations

Locale control:

- English
- German

The locale selection controls published Case Studies, Insights, and Pilot FAQs.

---

## 4. Admin Portal Navigation

Primary navigation:

- Dashboard
- Leads
- Companies
- Talent Applications
- Talent Profiles
- Interview Requests
- Managed Content
- Settings

### 4.1 Leads

Local views:

- Active
- Contact
- Pilot
- Calendly
- Archived

### 4.2 Companies

Local views:

- Active
- Invitation Pending
- On Pilot
- Archived

### 4.3 Talent Applications

Local views:

- Recruitment Pipeline
- Approved
- Profile Information Submitted
- Rejected
- Archived

### 4.4 Talent Profiles

Local views:

- Visible
- Hidden
- Archived

### 4.5 Managed Content

Secondary destinations:

- Trusted Logos
- Testimonials
- Services Hero Media
- Case Studies
- Insights
- Careers Roles
- Pilot FAQs

### 4.6 Settings

Secondary destinations:

- Email Templates
- Calendly

---

## 5. Client Workspace Navigation

Primary navigation:

- Dashboard
- Talent Directory
- Talent Pods
- Interview Requests
- Pilot Status

The workspace identity remains visible so the Client stays oriented to their Company.

Talent Pods organize visible Talent for planning only. They do not create
staffing assignments, reservations, interviews, or direct communication.

The Client Workspace does not contain company-user teams, user management,
contracts, billing, or messaging.

---

## 6. Talent Portal Navigation

The Talent Portal has one primary destination:

- Profile Management

Account-menu actions:

- Change Password
- Log Out

The Talent Portal does not display a dashboard or multi-section application sidebar. Page sections may use in-page anchors or a progress indicator when useful, but they remain part of Profile Management.

---

## 7. Shared Authenticated Navigation

Authenticated applications provide:

- Current user identity
- Account menu
- Change Password or password-reset entry where applicable
- Log Out

Users cannot navigate into applications their role does not permit.

---

## 8. Contextual Navigation

Contextual links connect related resources:

- Lead → converted Company
- Company → Client Workspace status and Interview Requests
- Talent Application → profile-completion submission and Talent Profile
- Talent Directory or Talent Profile → Add to Pod
- Talent Pod → member Talent Profiles
- Talent Profile → linked Talent Application
- Interview Request → Company and Talent Profile
- Dashboard widgets → filtered resource lists

Destructive or lifecycle actions remain contextual actions rather than navigation destinations.

---

## 9. Deep Linking

Stable, permission-aware detail URLs are required for:

- Lead
- Company
- Talent Application
- Talent Profile
- Interview Request
- Case Study
- Insight
- Careers Role

Invitation, completion, and reset URLs are tokenized and may not be reused after consumption.

---

## 10. Navigation States

- The active primary and secondary destination is visually clear.
- Loading preserves the application shell when possible.
- An unresolved workspace ID shows Workspace Not Found with only Go Back.
- An existing workspace unavailable to the signed-in account shows Access Denied with only Go Back.
- Expired sessions redirect directly to Login; there is no standalone Session Expired screen.
- Expired token links show a dedicated recovery action where one exists.
- Empty lists explain the absence of data and offer a permitted next action.
