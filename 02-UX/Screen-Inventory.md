# BlihOps V1 — Screen Inventory

## 1. Overview

This inventory identifies the pages, modals, drawers, dialogs, panels, sections, and system states required to design BlihOps V1.

It is derived from:

- Product Brief
- Product Requirements Document
- User Personas
- Information Architecture
- Navigation
- Consolidated User Flows

The inventory is the UI-design boundary for V1. A surface must trace to a product requirement or user flow; future products and excluded features are not included.

---

## 2. Surface Types

| Type | Definition |
|------|------------|
| Page | A route-level destination with its own primary purpose. |
| Modal | A focused form or task layered over the current context. |
| Drawer | A contextual detail or editor that preserves the parent view. |
| Dialog | A short confirmation, warning, or blocking decision. |
| Panel | A persistent or temporary supporting surface within an application shell. |
| Section | A substantial embedded surface within a page. |
| Popover | A compact contextual menu or selector. |
| Full-Screen State | A route-level loading, error, access, or completion state. |
| Inline State | A local empty, validation, success, loading, or error state. |

Entry points name the user-visible origin, not an implementation route.

---

## 3. Authentication, Invitations, and Global Access

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| AUTH-01 | Admin Login | Page | Admin | Admin Portal entry, protected Admin redirect | Authenticate an Admin and route to the Admin Dashboard. | PRD 4.1; Flow 4.1 |
| AUTH-02 | Client and Talent Login | Page | Client, Talent | Public Log In, protected application redirect | Authenticate an activated user and route by role. | PRD 4.1; Flow 4.4 |
| AUTH-03 | Forgot Password | Page | Admin, Client, Talent | Login pages | Accept an email address and request a reset link without revealing account existence. | PRD 4.5; Flow 4.5 |
| AUTH-04 | Reset Request Sent | Full-Screen State | Admin, Client, Talent | Forgot Password submission | Confirm that reset instructions will be sent when the account is eligible. | PRD 4.5; Flow 4.5 |
| AUTH-05 | Reset Password | Page | Admin, Client, Talent | Valid password-reset link | Validate the token and accept a new password. | PRD 4.5; Flow 4.5 |
| AUTH-06 | Invalid or Expired Reset Link | Full-Screen State | Admin, Client, Talent | Invalid, expired, or consumed reset link | Explain why reset cannot continue and offer a new request. | PRD 4.5; Flow 4.5 |
| AUTH-07 | Client Account Activation | Page | Client | Valid Client invitation link | Show Company context, collect a password, and activate the single Client account. | PRD 4.2; Flow 4.2 |
| AUTH-08 | Talent Account Activation | Page | Talent | Valid Talent account invitation | Collect a password and activate the account linked to an existing Talent Profile. | PRD 4.4; Flow 4.3 |
| AUTH-09 | Invalid or Expired Invitation | Full-Screen State | Client, Talent | Invalid, expired, replaced, or consumed invitation | Block activation and explain the available recovery path. | PRD 4.2, 4.4; Flows 4.2–4.3 |
| AUTH-10 | Account Already Activated | Full-Screen State | Client, Talent | Consumed invitation for an active account | Direct the user to Login instead of repeating activation. | Flows 4.2–4.3 |
| AUTH-11 | Account Menu | Popover | Admin, Client, Talent | Authenticated application header | Show identity, password action where applicable, and Log Out. | Navigation 7; Flow 4.6 |
| AUTH-12 | Session Expired | Dialog | Admin, Client, Talent | Protected mutation or expired session | Explain session expiry and send the user to Login without falsely saving work. | PRD 14; Flow 4.7 |
| AUTH-13 | Access Denied | Full-Screen State | Admin, Client, Talent | Unauthorized protected route | Prevent cross-role or cross-owner access without exposing private data. | PRD 3.2; Flow 4.7 |
| AUTH-14 | Protected Resource Not Found | Full-Screen State | Admin, Client, Talent | Missing or inaccessible resource URL | Explain that the resource is unavailable and provide a safe return route. | Navigation 10; Flow 4.7 |

---

## 4. Public Website

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| WEB-01 | Home | Page | Visitor | Website entry, logo, primary navigation | Present BlihOps positioning, trust signals, services, testimonials, and primary actions. | PRD 5.1, 9.2–9.3; Flow 3.1 |
| WEB-02 | Locale Selector | Popover | Visitor | Public header or locale control | Switch between English and German locale-aware content. | PRD 5.6; Flow 3.1 |
| WEB-03 | Services | Page | Visitor | Primary navigation, Home service links | Explain services and display the managed hero video with cover image. | PRD 5.1, 9.4; Flow 3.1 |
| WEB-04 | How We Work | Page | Visitor | Primary navigation, contextual marketing links | Explain the BlihOps engagement process. | PRD 5.1; Flow 3.1 |
| WEB-05 | About | Page | Visitor | Primary navigation | Present company background and credibility. | PRD 5.1; Flow 3.1 |
| WEB-06 | Case Studies List | Page | Visitor | Primary navigation, Home links | Display published Case Studies in the active locale. | PRD 5.6, 9.5; Flow 3.1 |
| WEB-07 | Case Study Detail | Page | Visitor | Case Studies List, shared link | Display one published localized Case Study. | PRD 5.6, 9.5; Flow 3.1 |
| WEB-08 | Insights List | Page | Visitor | Primary navigation, Home links | Display published Insights in the active locale. | PRD 5.6, 9.6; Flow 3.1 |
| WEB-09 | Insight Detail | Page | Visitor | Insights List, shared link | Display one published localized Insight. | PRD 5.6, 9.6; Flow 3.1 |
| WEB-10 | Careers Roles | Page | Visitor | Primary navigation | Display active English-only roles with featured emphasis. | PRD 5.6, 9.7; Flow 3.6 |
| WEB-11 | Careers Role Detail | Page | Visitor | Careers Roles | Display the full information for one active role. | PRD 9.7; Flow 3.6 |
| WEB-12 | No Open Roles | Inline State | Visitor | Empty Careers Roles result | Explain that no roles are active without presenting stale listings. | Flow 3.6 |
| WEB-13 | Pilot | Page | Visitor | Primary navigation, Request a Pilot actions | Explain the pilot, display localized FAQs, and host the request journey. | PRD 5.1, 9.8; Flows 3.1, 3.3 |
| WEB-14 | Pilot Request Form | Section | Visitor | Pilot page, Request a Pilot action | Collect company, contact, and pilot information. | PRD 5.3; Flow 3.3 |
| WEB-15 | Pilot Request Success | Full-Screen State | Visitor | Successful Pilot Request | Confirm submission and offer Book Discovery Call. | PRD 5.3; Flow 3.3 |
| WEB-16 | Book a Call | Modal / External Surface | Visitor | Header action, Pilot success, marketing CTAs | Open the configured Calendly booking experience. | PRD 5.4; Flow 3.4 |
| WEB-17 | Contact | Page | Visitor | Primary or footer navigation | Present contact information and the general inquiry form. | PRD 5.1–5.2; Flow 3.2 |
| WEB-18 | Contact Form | Section | Visitor | Contact page, contextual contact actions | Collect a general business inquiry. | PRD 5.2; Flow 3.2 |
| WEB-19 | Contact Submission Success | Full-Screen State | Visitor | Successful Contact submission | Confirm the Lead was received and provide a safe next action. | PRD 5.2; Flow 3.2 |
| WEB-20 | Join Talent Pool | Page | Visitor | Header/footer action, Careers or talent marketing links | Collect a Talent Application and required resume. | PRD 5.5; Flow 3.5 |
| WEB-21 | Talent Application Resume Upload | Section | Visitor | Join Talent Pool form | Validate and upload the required resume with progress and errors. | PRD 5.5, 12; Flow 3.5 |
| WEB-22 | Talent Application Confirmation | Full-Screen State | Visitor | Successful Talent Application | Confirm receipt without implying profile or account creation. | PRD 5.5; Flow 3.5 |
| WEB-23 | Privacy | Page | Visitor | Footer, consent copy | Present the privacy policy. | PRD 5.1 |
| WEB-24 | Terms | Page | Visitor | Footer, consent copy | Present terms of service. | PRD 5.1 |
| WEB-25 | Public Content Empty State | Inline State | Visitor | Empty Case Studies or Insights listing | Explain the absence of published content. | Flow 3.1 |
| WEB-26 | Public Content Load Error | Inline State | Visitor | Failed public content request | Offer retry without exposing draft content. | PRD 14; Flow 3.1 |
| WEB-27 | Public Page Not Found | Page | Visitor | Unknown or removed public URL | Provide recovery links to primary destinations. | Flow 3.1 |

---

## 5. Admin Shell and Dashboard

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| ADM-01 | Admin Application Shell | Page Shell | Admin | Successful Admin login | Provide primary navigation, identity, account actions, and content area. | IA 4.2; Navigation 4 |
| ADM-02 | Admin Dashboard | Page | Admin | Login, Admin navigation | Summarize Leads, Companies, recruitment, profiles, invitations, interviews, and recent activity. | PRD 6.1; Flow 12.1 |
| ADM-03 | Dashboard Metric Card | Section | Admin | Admin Dashboard | Show an operational count and link to its filtered resource list. | PRD 6.1; Flow 12.1 |
| ADM-04 | Recent Activity Feed | Section | Admin | Admin Dashboard | Show recent important actions and open related details. | PRD 6.1, 13; Flow 12.1 |
| ADM-05 | Admin Notifications | Panel | Admin | Admin header | Show new Leads, applications, submissions, activations, and Interview Requests. | PRD 11.1 |
| ADM-06 | Dashboard Widget Error | Inline State | Admin | Failed Dashboard widget | Retry a failed metric without blocking the rest of the Dashboard. | Flow 12.1 |

---

## 6. Admin Lead Management

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| LEAD-01 | Leads List | Page | Admin | Admin navigation, Dashboard widgets | Browse, search, filter, and paginate active or archived Leads. | PRD 6.2; Flows 5.1–5.5 |
| LEAD-02 | Lead Filters | Panel | Admin | Leads List | Filter by type, status, source, and active/archive view. | PRD 6.2; Flow 5.2 |
| LEAD-03 | Create Lead | Modal | Admin | Leads List | Manually create a Lead. | PRD 6.2; Flow 5.1 |
| LEAD-04 | Possible Duplicate Lead or Company | Dialog | Admin | Create Lead validation | Warn about a matching resource before allowing or blocking creation. | PRD 6.2; Flow 5.1 |
| LEAD-05 | Lead Detail | Page | Admin | Leads List, Dashboard, activity link | Show company/contact data, source, status, notes, related Company, and activity. | PRD 6.2; Flow 5.2 |
| LEAD-06 | Edit Lead | Drawer | Admin | Lead Detail | Update Lead and contact information without leaving context. | PRD 6.2; Flow 5.2 |
| LEAD-07 | Lead Status Selector | Popover | Admin | Lead Detail, Leads List row action | Move a Lead through allowed pipeline statuses. | PRD 6.2; Flow 5.3 |
| LEAD-08 | Invalid Lead Transition | Dialog | Admin | Disallowed status selection | Explain why the selected transition cannot occur. | Flow 5.3 |
| LEAD-09 | Internal Note Editor | Section | Admin | Lead Detail | Add a time-stamped internal note. | PRD 6.2; Flow 5.2 |
| LEAD-10 | Convert Lead to Company | Modal | Admin | Qualified Lead Detail | Review prefilled Company data and confirm conversion. | PRD 6.2; Flow 5.4 |
| LEAD-11 | Conversion Duplicate Block | Dialog | Admin | Convert Lead validation | Block duplicate Company creation and link the existing Company when allowed. | PRD 6.2; Flow 5.4 |
| LEAD-12 | Archive Lead Confirmation | Dialog | Admin | Lead Detail, Leads List row action | Confirm removal from the active pipeline. | PRD 6.2; Flow 5.5 |
| LEAD-13 | Restore Lead Confirmation | Dialog | Admin | Archived Leads | Restore the Lead to the defined active state. | PRD 6.2; Flow 5.5 |
| LEAD-14 | Leads Empty State | Inline State | Admin | Empty filtered Leads List | Explain the result and offer Clear Filters or Create Lead when permitted. | Flow 5.1 |

---

## 7. Admin Company and Client Access

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| COMP-01 | Companies List | Page | Admin | Admin navigation, Dashboard, converted Lead | Browse active, invitation-pending, on-pilot, or archived Companies. | PRD 6.3; Flows 6.1–6.6 |
| COMP-02 | Company Filters | Panel | Admin | Companies List | Filter by Company, status, invitation, and archive state. | PRD 6.3 |
| COMP-03 | Create Company | Modal | Admin | Companies List | Manually create a Company and its Client Workspace. | PRD 6.3; Flow 6.1 |
| COMP-04 | Duplicate Company Block | Dialog | Admin | Company creation or Lead conversion | Prevent duplicate Company and Workspace creation. | PRD 6.2–6.3; Flows 5.4, 6.1 |
| COMP-05 | Company Detail | Page | Admin | Companies List, converted Lead, Interview Request | Present Company overview and local detail navigation. | PRD 6.3; Flow 6.2 |
| COMP-06 | Company Overview | Section | Admin | Company Detail | Show and edit Company information and status summary. | PRD 6.3; Flow 6.2 |
| COMP-07 | Edit Company | Drawer | Admin | Company Overview | Update Company information. | PRD 6.3; Flow 6.2 |
| COMP-08 | Client Access | Section | Admin | Company Detail | Show the single Client account or pending invitation and Workspace access state. | PRD 4.2, 6.3; Flows 6.2–6.4 |
| COMP-09 | Invite Client | Modal | Admin | Client Access | Send the one allowed Client account invitation. | PRD 4.2, 6.3; Flow 6.3 |
| COMP-10 | Resend Client Invitation | Dialog | Admin | Pending invitation action | Replace the previous usable token and resend the invitation. | PRD 4.2; Flow 6.3 |
| COMP-11 | Invitation Delivery Error | Inline State | Admin | Failed Client invitation | Show that delivery failed without claiming the invitation was received. | Flow 6.3 |
| COMP-12 | Deactivate Client Access | Dialog | Admin | Active Client account | Confirm access removal and active-session invalidation. | PRD 6.3; Flow 6.4 |
| COMP-13 | Reactivate Client Access | Dialog | Admin | Inactive Client account | Restore Client Workspace access when Company state permits. | PRD 6.3; Flow 6.4 |
| COMP-14 | Company Pilot Status | Section | Admin | Company Detail | Display the current Pilot, timeline, milestones, and participating talent. | PRD 6.3, 6.6; Flow 6.5 |
| COMP-15 | Edit Pilot Status | Drawer | Admin | Company Pilot Status | Update Pilot status, milestones, timeline, and participating talent. | PRD 6.6; Flow 6.5 |
| COMP-16 | Company Interview Requests | Section | Admin | Company Detail | Show Interview Requests belonging to the Company. | PRD 6.3, 6.6; Flow 6.2 |
| COMP-17 | Company Notes and Activity | Section | Admin | Company Detail | Show internal notes and recorded lifecycle activity. | PRD 6.3, 13; Flow 6.2 |
| COMP-18 | Archive Company Confirmation | Dialog | Admin | Company Detail, Companies List | Explain access impact and confirm archival. | PRD 6.3; Flow 6.6 |
| COMP-19 | Restore Company Confirmation | Dialog | Admin | Archived Companies | Restore Company administration without automatically reactivating Client access. | PRD 6.3; Flow 6.6 |
| COMP-20 | Companies Empty State | Inline State | Admin | Empty filtered Companies List | Explain the result and offer Clear Filters or Create Company. | Flow 6.1 |

---

## 8. Talent Recruitment and Profile Creation

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| APP-01 | Talent Applications Pipeline | Page | Admin | Admin navigation, Dashboard | Browse applications by recruitment stage, outcome, or archive state. | PRD 6.4; Flows 7.1–7.8 |
| APP-02 | Application Filters | Panel | Admin | Talent Applications Pipeline | Search and filter by stage, status, and submission state. | PRD 6.4 |
| APP-03 | Talent Application Detail | Page | Admin | Applications Pipeline, Dashboard notification | Present candidate, recruitment, completion, profile, and activity context. | PRD 6.4; Flow 7.1 |
| APP-04 | Candidate Information | Section | Admin | Application Detail | Show original application data and contact information. | PRD 6.4; Flow 7.1 |
| APP-05 | Candidate File Viewer | Modal | Admin | Resume or uploaded file action | Securely preview or download an authorized candidate file. | PRD 12; Flow 7.1 |
| APP-06 | Recruitment Assessments | Section | Admin | Application Detail | Show assessment stages, outcomes, scores, and notes. | PRD 6.4; Flow 7.2 |
| APP-07 | Record Stage Assessment | Modal | Admin | Recruitment Assessments | Record required stage outcome, scores, and notes. | PRD 6.4; Flow 7.2 |
| APP-08 | Invalid Stage Transition | Dialog | Admin | Disallowed stage action | Explain missing requirements or invalid transition order. | PRD 6.4; Flow 7.2 |
| APP-09 | Application Notes and Activity | Section | Admin | Application Detail | Add internal notes and review lifecycle history. | PRD 6.4, 13; Flow 7.1 |
| APP-10 | Reject Talent Application | Dialog | Admin | Application Detail, assessment outcome | Require a rejection reason and confirm closure. | PRD 6.4; Flow 7.3 |
| APP-11 | Approve Talent Summary | Dialog | Admin | Eligible Application Detail | Review required assessment completion and confirm approval. | PRD 6.4; Flow 7.4 |
| APP-12 | Request Profile Completion | Modal | Admin | Approved Application Detail | Confirm recipient and send the first, tokenized final-information request. | PRD 4.3, 6.4; Flow 7.4 |
| APP-13 | Resend Profile-Completion Request | Dialog | Admin | Pending completion request | Replace the prior usable token and resend. | PRD 4.3; Flow 7.4 |
| APP-14 | Profile-Completion Status | Section | Admin | Application Detail | Show request, expiry, delivery, submission, and token state. | PRD 4.3, 6.4; Flows 7.4–7.5 |
| APP-15 | Final-Information Review | Page / Section | Admin | Submitted completion notification, Application Detail | Compare original application, assessments, and final information. | PRD 6.4; Flow 7.6 |
| APP-16 | Create Talent Profile | Page | Admin | Final-Information Review | Build the profile manually, including admin-only fields. | PRD 6.4–6.5; Flow 7.6 |
| APP-17 | Client-Visible Profile Preview | Page / Modal | Admin | Create or Edit Talent Profile | Preview exactly what a Client will see before saving or publishing. | PRD 6.5, 7.3; Flow 7.6 |
| APP-18 | Duplicate Profile Block | Dialog | Admin | Create Talent Profile submission | Prevent a second profile for the same approved application. | PRD 6.4–6.5; Flow 7.6 |
| APP-19 | Invite Talent Account | Modal | Admin | Created Talent Profile, Profile Detail | Send the second invitation after the profile exists. | PRD 4.4, 6.4–6.5; Flow 7.7 |
| APP-20 | Resend Talent Account Invitation | Dialog | Admin | Pending Talent account invitation | Replace the previous token and resend account activation. | PRD 4.4; Flow 7.7 |
| APP-21 | Archive Application Confirmation | Dialog | Admin | Application Detail, pipeline row action | Confirm archival and invalidate pending completion tokens. | PRD 6.4; Flow 7.8 |
| APP-22 | Restore Application Confirmation | Dialog | Admin | Archived Applications | Restore the application to its valid prior stage. | PRD 6.4; Flow 7.8 |
| APP-23 | Applications Empty State | Inline State | Admin | Empty filtered pipeline | Explain the result and offer Clear Filters. | Flow 7.1 |

### Tokenized Talent Completion

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| COMPFORM-01 | Talent Final-Information Form | Page | Approved Talent with token | Valid profile-completion link | Collect final professional information without creating an account. | PRD 4.3; Flow 7.5 |
| COMPFORM-02 | Final-Information Photo Upload | Section | Approved Talent with token | Final-Information Form | Validate and upload the proposed profile photo. | PRD 4.3, 12; Flow 7.5 |
| COMPFORM-03 | Final-Information Resume Upload | Section | Approved Talent with token | Final-Information Form | Validate and upload or replace the resume. | PRD 4.3, 12; Flow 7.5 |
| COMPFORM-04 | Invalid or Expired Completion Link | Full-Screen State | Approved Talent with token | Invalid, expired, replaced, or consumed link | Block the form and explain how to contact BlihOps. | PRD 4.3; Flow 7.5 |
| COMPFORM-05 | Final-Information Submission Success | Full-Screen State | Approved Talent with token | Successful form submission | Confirm Admin review is next and avoid implying account creation. | PRD 4.3; Flow 7.5 |

---

## 9. Admin Talent Profile Management

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| PROFILE-01 | Talent Profiles List | Page | Admin | Admin navigation, Dashboard | Browse, search, filter, sort, and paginate visible, hidden, or archived profiles. | PRD 6.5; Flow 8.1 |
| PROFILE-02 | Profile Filters | Panel | Admin | Talent Profiles List | Filter by visibility, status, role, skills, availability, and archive state. | PRD 6.5; Flow 8.1 |
| PROFILE-03 | Admin Talent Profile Detail | Page | Admin | Profiles List, Application, Interview Request | Show all professional, commercial, verification, visibility, account, and internal information. | PRD 6.5; Flow 8.1 |
| PROFILE-04 | Edit Talent Profile | Page | Admin | Admin Talent Profile Detail | Edit professional and admin-only profile fields. | PRD 6.5; Flow 8.2 |
| PROFILE-05 | Talent Profile Files | Section | Admin | Profile Detail or Edit | Preview and manage authorized photo and resume files. | PRD 6.5, 12; Flow 8.2 |
| PROFILE-06 | Client-Visible Preview | Page / Modal | Admin | Profile Detail or Edit | Verify the exact client-safe presentation. | PRD 7.3; Flow 8.2 |
| PROFILE-07 | Publish Profile Confirmation | Dialog | Admin | Complete hidden Profile | Confirm making the profile visible to Clients. | PRD 6.5; Flow 8.3 |
| PROFILE-08 | Incomplete Profile Requirements | Dialog / Panel | Admin | Failed publish validation | List fields that must be completed before publication. | PRD 6.5; Flow 8.3 |
| PROFILE-09 | Hide Profile Confirmation | Dialog | Admin | Visible Profile | Confirm removal from Client discovery and explain unavailable placeholders in existing Pods. | PRD 6.5; Flow 8.3 |
| PROFILE-10 | Talent Account Access | Section | Admin | Profile Detail | Show invitation, activation, and account-access state. | PRD 4.4, 6.5; Flow 7.7 |
| PROFILE-11 | Archive Profile Confirmation | Dialog | Admin | Profile Detail, Profiles List | Explain Client visibility and Talent Portal access impact. | PRD 6.5; Flow 8.4 |
| PROFILE-12 | Restore Profile Confirmation | Dialog | Admin | Archived Profiles | Restore the profile as hidden without automatic account reactivation. | PRD 6.5; Flow 8.4 |
| PROFILE-13 | Profile Activity | Section | Admin | Profile Detail | Distinguish Admin changes from Talent-originated changes. | PRD 13; Flows 8.2, 10.2 |
| PROFILE-14 | Profiles Empty State | Inline State | Admin | Empty filtered Profiles List | Explain the result and offer Clear Filters. | Flow 8.1 |

---

## 10. Admin Interview Oversight

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| INTERVIEW-01 | Interview Requests List | Page | Admin | Admin navigation, Dashboard, Company Detail | Browse and filter Interview Requests across Companies. | PRD 6.6; Flow 12.4 |
| INTERVIEW-02 | Interview Request Filters | Panel | Admin | Interview Requests List | Filter by status, Company, Talent, and date. | PRD 6.6 |
| INTERVIEW-03 | Admin Interview Request Detail | Page / Drawer | Admin | Requests List, Company Detail | Show Company, Talent, context, status, scheduling, notes, and activity. | PRD 6.6; Flow 12.4 |
| INTERVIEW-04 | Update Interview Request | Modal | Admin | Admin Interview Request Detail | Change status, scheduling information, and client-visible updates. | PRD 6.6; Flow 12.4 |
| INTERVIEW-05 | Invalid Interview Transition | Dialog | Admin | Disallowed status update | Explain why the request cannot move to the selected status. | Flow 12.4 |
| INTERVIEW-06 | Interview Internal Notes | Section | Admin | Admin Interview Request Detail | Record private operational notes. | PRD 6.6; Flow 12.4 |
| INTERVIEW-07 | Interview Requests Empty State | Inline State | Admin | Empty filtered Requests List | Explain the result and offer Clear Filters. | Flow 12.4 |

---

## 11. Managed Website Content

### 11.1 Content Navigation

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| CMS-01 | Managed Content Home | Page | Admin | Admin navigation | Introduce and link the seven bounded content areas. | PRD 9.1; Navigation 4.5 |
| CMS-02 | Shared Media Upload | Modal | Admin | Content image or video control | Validate, upload, preview, replace, or cancel a media asset. | PRD 9.9, 12; Flows 11.1–11.6 |
| CMS-03 | Content Save Error | Inline State | Admin | Failed content mutation | Preserve editor data and offer retry without reporting success. | PRD 9.9, 14 |

### 11.2 Trusted Logos

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| LOGO-01 | Trusted Logos List | Page | Admin | Managed Content Home | Display logos, active state, website, and display order. | PRD 9.2; Flow 11.1 |
| LOGO-02 | Create or Edit Trusted Logo | Modal | Admin | Trusted Logos List | Manage name, logo image, optional website, and active state. | PRD 9.2; Flow 11.1 |
| LOGO-03 | Trusted Logo Reorder Mode | Section | Admin | Trusted Logos List | Change and save public display order. | PRD 9.2; Flow 11.1 |
| LOGO-04 | Delete Trusted Logo Confirmation | Dialog | Admin | Trusted Logo row action | Confirm permanent deletion. | PRD 9.9; Flow 11.1 |
| LOGO-05 | Trusted Logos Empty State | Inline State | Admin | Empty Trusted Logos List | Offer creation of the first logo. | Flow 11.1 |

### 11.3 Testimonials

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| TESTIMONIAL-01 | Testimonials List | Page | Admin | Managed Content Home | Display text/video type, active state, order, and Primary selection. | PRD 9.3; Flow 11.2 |
| TESTIMONIAL-02 | Create or Edit Testimonial | Modal / Drawer | Admin | Testimonials List | Edit shared and type-specific testimonial fields. | PRD 9.3; Flow 11.2 |
| TESTIMONIAL-03 | Testimonial Reorder Mode | Section | Admin | Testimonials List | Change and save public display order. | PRD 9.3; Flow 11.2 |
| TESTIMONIAL-04 | Select Primary Testimonial | Dialog | Admin | Active Testimonial action | Atomically move the Primary designation to the selected item. | PRD 9.3; Flow 11.2 |
| TESTIMONIAL-05 | Primary Replacement Required | Dialog | Admin | Deactivate or delete current Primary | Require another active Primary before completing the action. | PRD 9.3; Flow 11.2 |
| TESTIMONIAL-06 | Delete Testimonial Confirmation | Dialog | Admin | Testimonial row action | Confirm deletion and protect the Primary invariant. | PRD 9.9; Flow 11.2 |
| TESTIMONIAL-07 | Testimonials Empty State | Inline State | Admin | Empty Testimonials List | Offer creation and explain the Primary requirement. | Flow 11.2 |

### 11.4 Services Hero Media

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| HERO-01 | Services Hero Media Editor | Page | Admin | Managed Content Home | Manage the singleton video, cover image, accessible label, and update metadata. | PRD 9.4; Flow 11.3 |
| HERO-02 | Hero Media Preview | Section | Admin | Services Hero Media Editor | Preview video playback and cover behavior before saving. | PRD 9.4; Flow 11.3 |
| HERO-03 | Hero Media Upload Error | Inline State | Admin | Failed or invalid media upload | Preserve current live media and explain correction. | Flow 11.3 |

### 11.5 Case Studies

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| CASE-01 | Case Studies List | Page | Admin | Managed Content Home | Browse Draft/Published and featured Case Studies. | PRD 9.5; Flow 11.4 |
| CASE-02 | Case Study Filters | Panel | Admin | Case Studies List | Filter by status, category, featured state, and search text. | PRD 9.5 |
| CASE-03 | Case Study Editor | Page | Admin | Create action, Case Study row | Edit English/German tabs and shared metadata in one record. | PRD 9.5; Flow 11.4 |
| CASE-04 | Locale Completion Indicator | Section | Admin | Case Study Editor | Show required-field completeness for English and German. | PRD 9.5; Flow 11.4 |
| CASE-05 | Case Study Preview | Page / Modal | Admin | Case Study Editor | Preview the selected locale before publication. | PRD 9.5; Flow 11.4 |
| CASE-06 | Case Study Publish Validation | Dialog / Panel | Admin | Publish action | List incomplete locale/shared fields or duplicate slugs. | PRD 9.5; Flow 11.4 |
| CASE-07 | Unpublish Case Study Confirmation | Dialog | Admin | Published Case Study action | Confirm removal of both locales from the public site. | PRD 9.5; Flow 11.4 |
| CASE-08 | Delete Case Study Confirmation | Dialog | Admin | Case Study action | Confirm permanent deletion. | PRD 9.9; Flow 11.4 |
| CASE-09 | Case Studies Empty State | Inline State | Admin | Empty filtered Case Studies List | Offer creation or Clear Filters. | Flow 11.4 |

### 11.6 Insights

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| INSIGHT-01 | Insights List | Page | Admin | Managed Content Home | Browse Draft/Published and featured Insights. | PRD 9.6; Flow 11.5 |
| INSIGHT-02 | Insight Filters | Panel | Admin | Insights List | Filter by status, category, featured state, and search text. | PRD 9.6 |
| INSIGHT-03 | Insight Editor | Page | Admin | Create action, Insight row | Edit English/German tabs and shared metadata in one record. | PRD 9.6; Flow 11.5 |
| INSIGHT-04 | Locale Completion Indicator | Section | Admin | Insight Editor | Show English and German completeness. | PRD 9.6; Flow 11.5 |
| INSIGHT-05 | Insight Preview | Page / Modal | Admin | Insight Editor | Preview the selected locale before publication. | PRD 9.6; Flow 11.5 |
| INSIGHT-06 | Insight Publish Validation | Dialog / Panel | Admin | Publish action | List incomplete locale/shared fields or duplicate slugs. | PRD 9.6; Flow 11.5 |
| INSIGHT-07 | Unpublish Insight Confirmation | Dialog | Admin | Published Insight action | Confirm removal of both locales from the public site. | PRD 9.6; Flow 11.5 |
| INSIGHT-08 | Delete Insight Confirmation | Dialog | Admin | Insight action | Confirm permanent deletion. | PRD 9.9; Flow 11.5 |
| INSIGHT-09 | Insights Empty State | Inline State | Admin | Empty filtered Insights List | Offer creation or Clear Filters. | Flow 11.5 |

### 11.7 Careers Roles

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| ROLE-01 | Careers Roles List | Page | Admin | Managed Content Home | Browse English-only roles with active and featured state. | PRD 9.7; Flow 11.6 |
| ROLE-02 | Create or Edit Careers Role | Page / Drawer | Admin | Roles List | Manage all role content, active state, and featured state. | PRD 9.7; Flow 11.6 |
| ROLE-03 | Deactivate Careers Role Confirmation | Dialog | Admin | Active role action | Confirm removing the role and its detail from the public site. | PRD 9.7; Flow 11.6 |
| ROLE-04 | Delete Careers Role Confirmation | Dialog | Admin | Role action | Confirm permanent deletion. | PRD 9.9; Flow 11.6 |
| ROLE-05 | Careers Roles Empty State | Inline State | Admin | Empty Roles List | Offer creation of the first role. | Flow 11.6 |

### 11.8 Pilot FAQs

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| FAQ-01 | Pilot FAQs List | Page | Admin | Managed Content Home | Display bilingual entries, active state, and order. | PRD 9.8; Flow 11.7 |
| FAQ-02 | Create or Edit Pilot FAQ | Modal / Drawer | Admin | Pilot FAQs List | Edit English and German question/answer pairs. | PRD 9.8; Flow 11.7 |
| FAQ-03 | FAQ Reorder Mode | Section | Admin | Pilot FAQs List | Change and save public display order. | PRD 9.8; Flow 11.7 |
| FAQ-04 | Incomplete FAQ Activation | Dialog | Admin | Activate incomplete FAQ | Identify the missing English or German content. | PRD 9.8; Flow 11.7 |
| FAQ-05 | Delete Pilot FAQ Confirmation | Dialog | Admin | FAQ row action | Confirm permanent deletion. | PRD 9.9; Flow 11.7 |
| FAQ-06 | Pilot FAQs Empty State | Inline State | Admin | Empty FAQ List | Offer creation of the first bilingual FAQ. | Flow 11.7 |

---

## 12. Admin Settings

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| SETTINGS-01 | Settings Home | Page | Admin | Admin navigation | Link Email Templates and Calendly settings. | PRD 10; Navigation 4.6 |
| SETTINGS-02 | Email Templates List | Page | Admin | Settings Home | List Client invitation, completion request, Talent invitation, reset, and interview templates. | PRD 10.1; Flow 12.2 |
| SETTINGS-03 | Email Template Editor | Page | Admin | Email Templates List | Edit subject/body and show allowed variables. | PRD 10.1; Flow 12.2 |
| SETTINGS-04 | Email Template Preview | Modal | Admin | Template Editor | Preview the rendered template with representative variables. | Flow 12.2 |
| SETTINGS-05 | Invalid Template Variables | Dialog / Panel | Admin | Failed template validation | Identify unknown or missing required variables. | PRD 10.1; Flow 12.2 |
| SETTINGS-06 | Calendly Settings | Page | Admin | Settings Home | Show connection, event mapping, webhook state, and last webhook. | PRD 10.2; Flow 12.3 |
| SETTINGS-07 | Calendly Connection Diagnostic | Section | Admin | Calendly Settings | Show a safe health result without exposing secrets. | PRD 10.2; Flow 12.3 |
| SETTINGS-08 | Calendly Configuration Error | Inline State | Admin | Failed health or save action | Explain the diagnostic failure and offer retry. | Flow 12.3 |

---

## 13. Client Workspace

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| CLIENT-01 | Client Workspace Shell | Page Shell | Client | Successful login or invitation activation | Provide Company context, primary navigation, identity, and account actions. | IA 4.3; Navigation 5 |
| CLIENT-02 | Client Dashboard | Page | Client | Login, Workspace navigation | Summarize Pilot, available Talent, active Pods, activity, and Interview Requests. | PRD 7.1; Flow 9.1 |
| CLIENT-03 | Client Notifications | Panel | Client | Workspace header | Show Interview Request, Pilot, and unavailable Pod-member updates. | PRD 11.2 |
| CLIENT-04 | Talent Directory | Page | Client | Workspace navigation, Dashboard | Browse active visible Talent Profiles. | PRD 7.2; Flow 9.2 |
| CLIENT-05 | Talent Search and Sort | Section | Client | Talent Directory | Search profile text and choose result ordering. | PRD 7.2; Flow 9.2 |
| CLIENT-06 | Talent Filters | Drawer / Panel | Client | Talent Directory | Filter by role, skills, experience, and availability. | PRD 7.2; Flow 9.2 |
| CLIENT-07 | No Matching Talent | Inline State | Client | Empty filtered Talent Directory | Explain the result and offer Clear Filters. | Flow 9.2 |
| CLIENT-08 | Client Talent Profile | Page | Client | Talent Directory, Interview Request | Display only client-safe professional, availability, verification, file, and rate information. | PRD 7.3; Flow 9.3 |
| CLIENT-09 | Authorized Resume Viewer | Modal / Page | Client | Client Talent Profile | Preview or download an authorized visible resume. | PRD 7.3, 12; Flow 9.3 |
| CLIENT-10 | Talent Profile Unavailable | Full-Screen State | Client | Hidden or archived profile URL | Explain that the profile is no longer available and return to Directory. | Flow 9.3 |
| CLIENT-11 | Request Interview | Modal | Client | Client Talent Profile | Add context and submit an Interview Request through BlihOps. | PRD 7.5; Flow 9.4 |
| CLIENT-12 | Duplicate Interview Warning | Dialog | Client | Equivalent open request submission | Prevent an accidental duplicate and link the existing request. | Flow 9.4 |
| CLIENT-13 | Interview Request Confirmation | Full-Screen State / Modal | Client | Successful Interview Request | Confirm creation and link to Interview Requests. | PRD 7.5; Flow 9.4 |
| CLIENT-14 | Client Interview Requests | Page | Client | Workspace navigation, confirmation, Dashboard | List only the Company’s Interview Requests and statuses. | PRD 7.5; Flow 9.5 |
| CLIENT-15 | Client Interview Request Detail | Page / Drawer | Client | Client Interview Requests | Show Talent summary, submitted context, status, scheduling, and visible updates. | PRD 7.5; Flow 9.5 |
| CLIENT-16 | No Interview Requests | Inline State | Client | Empty Client Interview Requests | Link to Talent Directory to begin discovery. | Flow 9.5 |
| CLIENT-17 | Pilot Status | Page | Client | Workspace navigation, Dashboard | Show current Pilot, timeline, milestones, participating talent, and updates. | PRD 7.6; Flow 9.6 |
| CLIENT-18 | Pre-Pilot State | Inline State | Client | Pilot Status without active Pilot | Explain the current state before a Pilot begins. | Flow 9.6 |
| CLIENT-19 | Client Workspace Unavailable | Full-Screen State | Client | Deactivated account or archived Company | Explain that access is unavailable and direct the Client to BlihOps. | PRD 7.7; Flow 9.1 |
| CLIENT-20 | Talent Pods | Page | Client | Workspace navigation, Dashboard | Browse active planning Pods belonging to the Client Workspace. | PRD 7.4; Flow 9.7 |
| CLIENT-21 | No Talent Pods | Inline State | Client | Empty Talent Pods page | Explain planning-only Pods and offer Create Pod. | Flow 9.7 |
| CLIENT-22 | Create Talent Pod | Modal | Client | Talent Pods page | Create a Pod with a required name and optional planning note. | PRD 7.4; Flow 9.7 |
| CLIENT-23 | Talent Pod Detail | Page | Client | Talent Pods page, Dashboard activity | Review Pod purpose, members, optional Pod Lead, and recent changes. | PRD 7.4; Flow 9.7 |
| CLIENT-24 | Edit Talent Pod | Modal / Drawer | Client | Talent Pod Detail | Rename the Pod or update its planning note. | PRD 7.4; Flow 9.7 |
| CLIENT-25 | Add Pod Members | Drawer / Modal | Client | Talent Pod Detail | Find and add active visible Talent without duplicating membership. | PRD 7.4; Flow 9.7 |
| CLIENT-26 | Add Talent to Pod | Popover / Modal | Client | Talent Directory card, Client Talent Profile | Select an existing Pod or create a new Pod for the chosen Talent. | PRD 7.2–7.4; Flow 9.7 |
| CLIENT-27 | Select Pod Lead | Popover / Dialog | Client | Talent Pod Detail | Select or clear one optional Pod Lead from current members. | PRD 7.4; Flow 9.7 |
| CLIENT-28 | Remove Pod Member | Dialog | Client | Talent Pod Detail | Confirm member removal and explain when the Pod Lead will be cleared. | PRD 7.4; Flow 9.7 |
| CLIENT-29 | Archive Talent Pod | Dialog | Client | Talent Pod Detail | Confirm archival without implying any Talent or Pilot status change. | PRD 7.4; Flow 9.7 |
| CLIENT-30 | Unavailable Pod Member | Inline State | Client | Talent Pod Detail with a hidden or archived member | Preserve planning context while blocking access to an unavailable profile. | PRD 7.4; Flow 9.7 |

---

## 14. Talent Portal

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| TALENT-01 | Talent Portal Shell | Page Shell | Talent | Successful login or account activation | Provide identity and account actions around the single profile destination. | IA 4.4; Navigation 6 |
| TALENT-02 | Profile Management | Page | Talent | Login, Talent Portal entry | View and edit permitted fields on the Talent’s own profile. | PRD 8.1–8.3; Flows 10.1–10.2 |
| TALENT-03 | Professional Information | Section | Talent | Profile Management | Edit photo, headline, bio, and primary role. | PRD 6.5, 8.2; Flow 10.2 |
| TALENT-04 | Skills and Experience | Section | Talent | Profile Management | Edit tech stack, secondary skills, and years of experience. | PRD 6.5, 8.2; Flow 10.2 |
| TALENT-05 | Links and Resume | Section | Talent | Profile Management | Edit portfolio/social links and replace the resume. | PRD 6.5, 8.2; Flows 10.2–10.3 |
| TALENT-06 | Availability | Section | Talent | Profile Management | Edit availability, earliest start, and preferred engagement. | PRD 6.5, 8.2; Flow 10.2 |
| TALENT-07 | Profile Photo Upload | Modal | Talent | Professional Information | Validate, upload, preview, or cancel a photo replacement. | PRD 8.2, 12; Flow 10.3 |
| TALENT-08 | Resume Upload | Modal | Talent | Links and Resume | Validate, upload, replace, or cancel a resume. | PRD 8.2, 12; Flow 10.3 |
| TALENT-09 | Profile Save Success | Inline State | Talent | Successful Profile Management save | Confirm changes are saved and immediately live where visible. | PRD 8.3; Flow 10.2 |
| TALENT-10 | Profile Validation Error | Inline State | Talent | Invalid save | Identify fields requiring correction without losing valid work. | Flow 10.2 |
| TALENT-11 | Concurrent Profile Update | Dialog | Talent | Conflict with a newer Admin update | Require refresh or conflict resolution before overwriting fields. | Flow 10.2 |
| TALENT-12 | Talent Profile Access Blocked | Full-Screen State | Talent | Missing linkage, archived profile, or disabled access | Block editing and direct the Talent to BlihOps. | PRD 8.3; Flows 4.4, 10.1 |

---

## 15. Shared Operational States and Reusable Surfaces

| ID | Surface | Type | Actor | Entry Points | Purpose | Source |
|----|---------|------|-------|--------------|---------|--------|
| SYS-01 | Route Loading | Full-Screen State | Admin, Client, Talent, Visitor | Initial protected or public route load | Communicate loading while preventing premature interaction. | PRD 14 |
| SYS-02 | List Loading Skeleton | Inline State | Admin, Client | Resource list load | Preserve layout while table or card results load. | PRD 14 |
| SYS-03 | Mutation in Progress | Inline State | Admin, Client, Talent, Visitor | Form or action submission | Prevent duplicate actions and communicate progress. | User Flows 2.2 |
| SYS-04 | Save Success | Inline State | Admin, Client, Talent | Successful mutation | Confirm the specific saved action truthfully. | User Flows 2.2 |
| SYS-05 | Retryable Save Error | Inline State | Admin, Client, Talent, Visitor | Failed mutation | Preserve safe input, explain failure, and offer retry. | PRD 14; User Flows 2.2 |
| SYS-06 | Delete Confirmation | Dialog | Admin | Destructive content action | Name the target, explain impact, and require explicit confirmation. | PRD 9.9; User Flows 2.2 |
| SYS-07 | Upload Progress | Inline State | Admin, Talent, Visitor | File or media upload | Show file identity, progress, cancellation, and completion. | PRD 12, 14 |
| SYS-08 | Upload Validation Error | Inline State | Admin, Talent, Visitor | Invalid file type or size | Explain the accepted constraints without discarding other form values. | PRD 12; User Flows 2.2 |
| SYS-09 | Empty Filtered Result | Inline State | Admin, Client | Search or filters return nothing | Explain the state and offer Clear Filters. | User Flows 2.2 |
| SYS-10 | Stale Data Conflict | Dialog | Admin, Talent | Concurrent update | Prevent silent overwrite and require refresh or explicit resolution. | Flows 5.2, 8.2, 10.2 |

---

## 16. Explicitly Excluded V1 Surfaces

The following surfaces must not be designed for V1:

- Public registration
- Talent Dashboard
- Talent-created initial profile
- Client user-management or additional-user invitation
- Company-user Teams or additional Client-user management
- Pod staffing, reservation, assignment, contracting, or direct-contact actions
- Direct Client–Talent messaging
- Contracts, payments, or invoicing
- Public Talent Marketplace
- BlihOps Skills
- AI matching
- General-purpose page builder
- Advanced analytics or reporting

---

## 17. UI Design Handoff Rules

- Design route-level Pages and Full-Screen States first.
- Design reusable Dialog, upload, validation, loading, and feedback patterns once, then apply them consistently.
- Keep the two talent invitations visually and verbally distinct:
  - Profile-completion request: tokenized final-information form, no account.
  - Talent account invitation: password activation after the Admin-created profile exists.
- Display Talent Pods as planning-only resources; never present them as staffing
  assignments or company-user Teams.
- Never display a Talent Dashboard.
- Client Profile designs must omit all internal and unauthorized Talent fields.
- Talent Profile Management must not expose editable rates, assessments, verification, visibility, status, or internal notes.
- Bilingual content editors must show English and German completion before publication.
- Responsive and accessible states are required for every surface, not separate optional screens.
