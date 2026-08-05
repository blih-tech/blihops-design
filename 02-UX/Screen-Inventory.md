# BlihOps V1 Screen Inventory

## 1. Purpose and Scope

This inventory lists the main V1 screens and screen families that will be
considered for wireframing. It connects the Information Architecture to the
User Flows without defining visual layout or detailed interaction behavior.

Each row represents a route-level screen, a major detail screen, or a repeated
screen family. Loading, empty, validation, success, error, expired-link, and
access-denied states are variants of the relevant screen unless noted otherwise.

## 2. Shared Authentication and Entry Routes

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| AUTH-01 | BlihOps Web Login | Client / Talent | Authenticate and redirect by role | Public Web → Login |
| AUTH-02 | Admin Login | Admin | Authenticate into BlihOps Admin | Admin entry |
| AUTH-03 | Accept Invitation | Client / Talent | Create a password and activate the invited account | Email → `/accept-invitation` |
| AUTH-04 | Password Reset Request | Client / Talent / Admin | Request a password-reset email | Login → Forgot Password |
| AUTH-05 | Set New Password | Client / Talent / Admin | Set a new password from a valid reset link | Password-reset email |
| AUTH-06 | Link and Access Status | Client / Talent / Admin | Explain invalid, expired, consumed, or unauthorized access | Invalid link or protected route |

`AUTH-01` is one shared Web login screen. The authenticated role determines the
destination: Client Workspace or Talent Profile Management. `AUTH-03` is also a
single shared screen; its token identifies the account type.

## 3. Public Website Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| WEB-01 | Home | Visitor | Introduce BlihOps and guide visitors to key actions | Public Web |
| WEB-02 | Services | Visitor | Explain the service offering and delivery model | Public navigation |
| WEB-03 | How We Work | Visitor | Explain the BlihOps process | Public navigation |
| WEB-04 | About | Visitor | Present company and trust context | Public navigation |
| WEB-05 | Case Studies Index | Visitor | Browse published Case Studies | Public navigation |
| WEB-06 | Case Study Detail | Visitor | Read one published Case Study | Case Studies Index |
| WEB-07 | Insights Index | Visitor | Browse published Insights | Public navigation |
| WEB-08 | Insight Detail | Visitor | Read one published Insight | Insights Index |
| WEB-09 | Careers Index | Visitor | Browse active Career roles | Public navigation |
| WEB-10 | Career Role Detail | Visitor | Review one active role | Careers Index |
| WEB-11 | Pilot | Prospective Client | Understand and request a Pilot; access FAQs and booking | Public navigation |
| WEB-12 | Contact | Prospective Client | Submit a general business inquiry | Public navigation |
| WEB-13 | Join Talent Pool | Candidate | Submit the Round 1 Talent Pool application (assessment fields and resume) | Public navigation |
| WEB-14 | Privacy | Visitor | Read privacy information | Public footer |
| WEB-15 | Terms | Visitor | Read terms information | Public footer |
| WEB-16 | Complete Profile | Candidate | Submit remaining profile-building fields after receiving a completion request token | Email → `/complete-profile` |

Case Studies, Insights, and Pilot content use the selected locale. Only
published or active content is visible publicly.

## 4. Protected Client Workspace Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| CLIENT-01 | Workspace Dashboard | Client | Show Company context, Pilot, Talent, Pods, requests, and activity | Successful login or invitation activation |
| CLIENT-02 | Talent Directory | Client | Search, filter, sort, paginate, and browse visible Talent | Workspace navigation |
| CLIENT-03 | Talent Profile | Client | Review one client-visible Talent Profile | Directory, Pod, or related request |
| CLIENT-04 | Talent Pods | Client | View and manage the Company's planning Pods | Workspace navigation |
| CLIENT-05 | Pod Detail | Client | Manage Pod name, note, members, and optional Pod Lead | Talent Pods |
| CLIENT-06 | Interview Requests | Client | View the Company's request history and statuses | Workspace navigation or confirmation |
| CLIENT-07 | Pilot Status | Client | View current Pilot status, milestones, Talent, and updates | Workspace navigation |

Pods are planning-only. They do not reserve, assign, hire, notify, or contact
Talent. No additional Client-user management screen exists in V1.

## 5. Protected Talent Portal Screens

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| TALENT-01 | Profile Management | Approved Talent | View and edit permitted profile fields, photo, resume, links, and availability | Successful login or invitation activation |

There is no Talent Dashboard. Talent lands directly on Profile Management and
cannot edit Admin-managed fields.

## 6. Admin Portal Screens

### 6.1 Operations

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| ADMIN-01 | Admin Dashboard | Admin | Summarize Leads, Applications, Companies, Profiles, requests, and activity | Successful Admin login |
| ADMIN-02 | Leads | Admin | Search, filter, paginate, create, and manage Leads | Admin navigation or Dashboard |
| ADMIN-03 | Lead Detail | Admin | Review Lead information, notes, status, and conversion action | Leads |
| ADMIN-04 | Companies | Admin | Search, filter, and open Companies | Admin navigation or Lead conversion |
| ADMIN-05 | Company Detail | Admin | Manage Company, Workspace access, Pilot, Interview Requests, notes, and activity | Companies or Lead conversion |
| ADMIN-06 | Talent Applications | Admin | Search, filter, paginate, and manage recruitment applications | Admin navigation or Dashboard |
| ADMIN-07 | Application Detail | Admin | Review candidate data, files, stages, assessments, notes, and invitations | Talent Applications |
| ADMIN-08 | Talent Profiles | Admin | Search, filter, sort, paginate, and manage profiles | Admin navigation or Dashboard |
| ADMIN-09 | Talent Profile Detail | Admin | Create Profile/account from an approved application with submitted completion data; edit, publish, hide, archive, restore, and invite Talent | Talent Profiles or Application Detail |

Interview Requests are contained within `ADMIN-05 Company Detail`; there is no
separate Admin Interview Requests screen. Pilot status and milestones also remain
within Company Detail.

### 6.2 Managed Website Content

| ID | Screen family | User | Purpose | Primary entry |
|---|---|---|---|---|
| ADMIN-10 | Trusted Logos Manager | Admin | Create, edit, order, activate, deactivate, and delete logo entries | Admin → Content |
| ADMIN-11 | Testimonials Manager | Admin | Manage text/video testimonials and the primary selection | Admin → Content |
| ADMIN-12 | Services Hero Media Editor | Admin | Replace the global Services video, cover, and accessible label | Admin → Content |
| ADMIN-13 | Case Studies Manager | Admin | List and create/edit bilingual Case Studies | Admin → Content |
| ADMIN-14 | Insights Manager | Admin | List and create/edit bilingual Insights | Admin → Content |
| ADMIN-15 | Careers Roles Manager | Admin | List and create/edit English-only Career roles | Admin → Content |
| ADMIN-16 | Pilot FAQs Manager | Admin | List and create/edit bilingual Pilot FAQs | Admin → Content |

The manager screen families include their relevant list, create, and edit
states. Publication, activation, ordering, deletion confirmation, upload
failure, and bilingual validation are variants within those families.

### 6.3 Settings

| ID | Screen | User | Purpose | Primary entry |
|---|---|---|---|---|
| ADMIN-17 | Email Templates | Admin | View and edit validated notification templates | Admin → Settings |
| ADMIN-18 | Calendly Settings | Admin | Review connection, webhook, event mapping, and last received event | Admin → Settings |

## 7. Inventory Conventions

- `WEB-*` identifies public BlihOps Web screens.
- `AUTH-*` identifies shared authentication and access screens.
- `CLIENT-*` and `TALENT-*` identify protected Web destinations.
- `ADMIN-*` identifies BlihOps Admin destinations.
- Company Detail is the context for Client access, Pilot information, Interview
  Requests, notes, and activity.
- Drawers, modals, confirmation prompts, and inline forms are documented as
  variants of their parent screen unless they become a distinct route-level
  destination during wireframing.
- The API is not included because it has no user-facing screens.
