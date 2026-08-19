# BlihOps V1 Screen Inventory

## 1. Purpose and Scope

This inventory lists the main V1 screens and screen families considered for
wireframing. It connects the Information Architecture to the User Flows without
defining visual layout.

Loading, empty, validation, success, error, expired-link, unavailable, and
access-denied states are variants of the relevant screen unless noted otherwise.

The Client Workspace is a Web experience scoped by Company ID. There is no
Workspace ID route or Workspace database entity in V1.

## 2. Shared Authentication and Entry Routes

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| AUTH-01 | BlihOps Web Login | Client / Talent | Authenticate and redirect by role | Public Web → Login |
| AUTH-02 | Admin Login | Admin | Authenticate into BlihOps Admin | Admin entry |
| AUTH-03 | Accept Invitation | Client / Talent | Create password and activate invited account | Invitation email |
| AUTH-04 | Password Reset Request | Client / Talent / Admin | Request password reset | Login → Forgot Password |
| AUTH-05 | Set New Password | Client / Talent / Admin | Set password from valid reset link | Reset email |
| AUTH-06 | Link and Access Status | Client / Talent / Admin | Explain invalid, expired, consumed, deactivated, or archived access | Link or protected route |

Successful invitation activation redirects:

```text
CLIENT → /client-workspace
TALENT → Talent Profile Management
```

## 3. Public Website Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| WEB-01 | Home | Visitor | Introduce Blihops and guide key actions | Public Web |
| WEB-02 | Services | Visitor | Explain service offering | Public navigation |
| WEB-03 | How We Work | Visitor | Explain the process | Public navigation |
| WEB-04 | About | Visitor | Present company and trust context | Public navigation |
| WEB-05 | Case Studies Index | Visitor | Browse published Case Studies | Public navigation |
| WEB-06 | Case Study Detail | Visitor | Read one Case Study | Case Studies Index |
| WEB-07 | Insights Index | Visitor | Browse published Insights | Public navigation |
| WEB-08 | Insight Detail | Visitor | Read one Insight | Insights Index |
| WEB-09 | Careers Index | Visitor | Browse active roles | Public navigation |
| WEB-10 | Career Role Detail | Visitor | Review one role | Careers Index |
| WEB-11 | Public Pilot | Prospective Client | Learn about and request a Pilot; access FAQs and booking | Public navigation |
| WEB-12 | Contact | Prospective Client | Submit a general inquiry | Public navigation |
| WEB-13 | Join Talent Pool | Candidate | Submit Round 1 application and resume | Public navigation |
| WEB-14 | Privacy | Visitor | Read privacy information | Public footer |
| WEB-15 | Terms | Visitor | Read terms information | Public footer |
| WEB-16 | Complete Profile | Candidate | Submit Round 2 completion data from token link | Completion email |

The Public Pilot creates a Lead. It is not the authenticated Client Pilot
Request screen.

## 4. Protected Client Workspace Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| CLIENT-01 | Work | Client | Show preparation, Pilot, completed history, or Active partnership state | Login or invitation activation |
| CLIENT-02 | Talent Directory | Client | Search, filter, sort, paginate, and browse visible Talent | Workspace navigation |
| CLIENT-03 | Talent Detail | Client | Review a client-visible Talent Profile and request interview/add to Pod | Directory or Pod |
| CLIENT-04 | Private Pods | Client | View and manage private planning Pods | Workspace navigation |
| CLIENT-05 | Pod Detail | Client | Manage title, description, members, Pod Lead, and actions | Private Pods |
| CLIENT-06 | Interview Requests | Client | View individual and grouped request history and statuses | Workspace navigation |
| CLIENT-07 | Pilot Request Form | Client | Submit the one free Pilot Request during Pilot Preparation | Work |
| CLIENT-08 | Pilot Request Review | Client | View submitted/in-review/rejected request and resubmit when allowed | Work |
| CLIENT-09 | Planned Pilot | Client | View approved Pilot before Admin starts it | Work |
| CLIENT-10 | Active Pilot Work | Client | View official Pilot information and submit progress updates | Work |
| CLIENT-11 | Completed Pilot Detail | Client | View completed Pilot read-only | Work → Pilot history |
| CLIENT-12 | Active Partnership | Client | View simple partnership status and Pilot history | Work |

Client Pods are private. Admin cannot see Pod details. No client contract, task,
billing, milestone, chat, or daily work management screens exist in V1.

## 5. Protected Talent Portal Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| TALENT-01 | Profile Management | Approved Talent | View and edit permitted profile fields | Login or invitation activation |

There is no Talent Dashboard.

## 6. Admin Portal Screens

### 6.1 Operations

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| ADMIN-01 | Admin Dashboard | Admin | Summarize Leads, Applications, Companies, requests, Pilots, and activity | Admin login |
| ADMIN-02 | Leads | Admin | Search, filter, paginate, and manage Leads | Admin navigation |
| ADMIN-03 | Lead Detail | Admin | Review Lead, notes, status, and conversion action | Leads |
| ADMIN-04 | Companies | Admin | Search, filter, and open Companies | Admin navigation or Lead conversion |
| ADMIN-05 | Company Detail | Admin | Manage Company, Client Account, invitation, access, requests, Pilots, notes, and activity | Companies |
| ADMIN-06 | Talent Applications | Admin | Search, filter, paginate, and manage applications | Admin navigation |
| ADMIN-07 | Application Detail | Admin | Review candidate data, files, stages, assessments, notes, and invitations | Talent Applications |
| ADMIN-08 | Talent Profiles | Admin | Search, filter, sort, paginate, and manage profiles | Admin navigation |
| ADMIN-09 | Talent Profile Detail | Admin | Create/edit/publish/hide/archive Profile and manage Talent invitation | Talent Profiles or Application |

Company Detail contains:

- Company profile
- Client Account and invitation status
- Derived Workspace access state
- Interview Requests
- Pilot Requests
- Pilot detail and history
- Internal notes
- Internal activity

There is no Admin Pod screen. Admin sees only individual Interview Requests and
Pilot participants resulting from Client actions.

### 6.2 Managed Website Content

| ID | Screen family | User | Purpose |
|---|---|---|---|
| ADMIN-10 | Trusted Logos Manager | Admin | Manage logo entries |
| ADMIN-11 | Testimonials Manager | Admin | Manage text/video testimonials and primary selection |
| ADMIN-12 | Services Hero Media Editor | Admin | Manage global Services media |
| ADMIN-13 | Case Studies Manager | Admin | Manage bilingual Case Studies |
| ADMIN-14 | Insights Manager | Admin | Manage bilingual Insights |
| ADMIN-15 | Careers Roles Manager | Admin | Manage English-only Careers roles |
| ADMIN-16 | Pilot FAQs Manager | Admin | Manage bilingual Pilot FAQs |

### 6.3 Settings

| ID | Screen | User | Purpose |
|---|---|---|---|
| ADMIN-17 | Email Templates | Admin | Manage essential transactional email templates |
| ADMIN-18 | Cal.com Settings | Admin | Review connection and webhook configuration |

## 7. Screen State Requirements

### Client Work

- Pilot Preparation empty state with Submit Pilot Request
- Request Submitted/In Review state
- Rejected request with edit/resubmit action
- Planned Pilot state
- Active Pilot state with Add Progress Update
- Completed Pilot read-only state
- Active Partnership state with simple stats and history
- Archived Company unavailable state

### Company Detail

- Pending invitation
- Active Client Account
- Deactivated Client Account
- Active workspace access
- Blocked because Company is archived
- Pilot request review/rejection/approval
- Planned, active, completed, and cancelled Pilot states

### Shared

- Loading
- Empty
- Validation failure
- Retryable mutation failure
- Expired invitation
- Consumed invitation
- Unauthorized resource
- Archived or unavailable resource

## 8. Inventory Conventions

- `AUTH-*` identifies shared authentication/access screens.
- `WEB-*` identifies public Blihops Web screens.
- `CLIENT-*` and `TALENT-*` identify protected Web screens.
- `ADMIN-*` identifies Blihops Admin screens.
- Drawers, dialogs, confirmation prompts, and inline forms are variants unless
  they become distinct route-level destinations.
- The API is not included because it has no user-facing screens.
