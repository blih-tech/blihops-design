# BlihOps V1 Screen Inventory

## 1. Purpose

This document lists the foundational screens required for the BlihOps V1 Web
and Admin applications.

It covers:

- Client and Talent authentication
- Talent profile completion before account creation
- Client Workspace
- Talent Portal
- Admin authentication
- Admin operations
- Managed website content
- Admin settings

The public marketing website is outside this inventory. Loading, validation,
empty, error, upload, confirmation, and responsive states will be designed
within their related screens rather than listed as separate inventory items.

## 2. Authentication and Access

| ID | Screen | Purpose |
|---|---|---|
| AUTH-01 | Client and Talent Sign In | Authenticate an activated Client or Talent account and route the user to the correct protected experience. |
| AUTH-02 | Accept Client Invitation | Let the single invited Client representative create a password and activate access to the Company's Workspace. |
| AUTH-03 | Accept Talent Invitation | Let Talent create a password and activate an account after Admin has created the Talent Profile. |
| AUTH-04 | Forgot Password | Accept an email address and request password-reset instructions without revealing whether an account exists. |
| AUTH-05 | Reset Email Sent | Confirm that reset instructions will be sent when the account is eligible. |
| AUTH-06 | Reset Password | Let a user create a new password with a valid reset token. |
| AUTH-07 | Invalid Reset Token | Explain that the reset link is invalid, expired, or already used and provide a safe recovery action. |
| AUTH-08 | Password Reset Complete | Confirm the password was reset and direct the user to Sign In. |

Invalid, expired, replaced, and already-used invitation states are handled
within the related invitation screen.

## 3. Talent Profile Completion

The profile-completion journey happens before the Talent account exists. It is
opened through a secure, expiring token and is not an authenticated Talent
Portal screen.

| ID | Screen | Purpose |
|---|---|---|
| COMPLETE-01 | Talent Profile Completion Form | A multi-step form that collects all Talent-provided information required for Admin to create the Talent Profile. |
| COMPLETE-02 | Profile Completion Submitted | Confirm that the information was submitted for Admin review without implying that a profile or account was created automatically. |

The multi-step form includes:

- Personal and contact information required from the Talent
- Profile photo
- Professional headline and biography
- Primary role
- Technology stack and additional skills
- Years of experience
- Portfolio, GitHub, and LinkedIn links
- Availability
- Earliest start date
- Preferred engagement
- Review and final submission

The original Talent Pool application resume is reused and is not requested
again. Admin-controlled fields are excluded, including rates, seniority,
assessment information, verification, client visibility, lifecycle status,
and internal notes.

Invalid, expired, replaced, or consumed completion links are handled as a
blocked state within this journey.

## 4. Client Workspace

| ID | Screen | Purpose |
|---|---|---|
| CLIENT-01 | Client Workspace Shell | Provide Company context, primary navigation, account access, and the shared protected layout. |
| CLIENT-02 | Dashboard | Give the Client a high-level view of available Talent, Pods, Interview Requests, Pilot progress, and recent activity. |
| CLIENT-03 | Talent Directory | List visible Talent using Talent cards with search, filters, sorting, and pagination. |
| CLIENT-04 | Talent Profile | Show the selected Talent's approved client-visible professional information, availability, verification, files, and commercial information. |
| CLIENT-05 | Pods | List the Client Workspace's active planning Pods. |
| CLIENT-06 | Pod Details | Show a Pod's name, planning purpose, members, optional Pod Lead, and available management actions. |
| CLIENT-07 | Interview Requests | List the Company's Interview Requests and their current statuses. |
| CLIENT-08 | Interview Request Details | Show the selected request's Talent summary, submitted context, scheduling information, status, and client-visible updates. |
| CLIENT-09 | Pilot Status | Show the current Pilot, timeline, milestones, participating Talent, and updates. |

Request Interview, create or edit Pod, add or remove Pod members, select a Pod
Lead, archive a Pod, notifications, and account actions are designed as actions
or overlays within the related foundation screens.

Talent Pods are private planning groups. They must never be presented as
company-user teams, staffing assignments, reservations, hiring commitments, or
direct communication with Talent.

## 5. Talent Portal

| ID | Screen | Purpose |
|---|---|---|
| TALENT-01 | Talent Portal Shell | Provide identity, account actions, and the shared protected layout for Talent. |
| TALENT-02 | Profile Management | Let Talent view and update the permitted information on their own Admin-created Talent Profile. |

Profile Management includes:

- Profile photo
- Professional headline and biography
- Primary role
- Technology stack and additional skills
- Years of experience
- Portfolio, GitHub, and LinkedIn links
- Resume replacement
- Availability
- Earliest start date
- Preferred engagement

Talent cannot edit rates, seniority, assessments, verification, client
visibility, lifecycle status, or internal notes. The Talent Portal has no
dashboard.

## 6. Admin Authentication

| ID | Screen | Purpose |
|---|---|---|
| ADMIN-AUTH-01 | Admin Sign In | Authenticate an internally provisioned Admin and open the Admin Dashboard. |
| ADMIN-AUTH-02 | Forgot Password | Accept an Admin email address and request password-reset instructions without revealing whether the account exists. |
| ADMIN-AUTH-03 | Reset Email Sent | Confirm that reset instructions will be sent when the Admin account is eligible. |
| ADMIN-AUTH-04 | Reset Password | Let an Admin create a new password with a valid reset token. |
| ADMIN-AUTH-05 | Invalid Reset Token | Explain that the reset link is invalid, expired, or already used and provide a safe recovery action. |
| ADMIN-AUTH-06 | Password Reset Complete | Confirm the password was reset and direct the Admin to Sign In. |

## 7. Admin Application

| ID | Screen | Purpose |
|---|---|---|
| ADMIN-01 | Admin Application Shell | Provide primary navigation, Admin identity, account actions, notifications, and the shared operational layout. |
| ADMIN-02 | Dashboard | Summarize Leads, Companies, Talent Applications, Talent Profiles, pending invitations and submissions, Interview Requests requiring attention, and recent activity. |
| ADMIN-03 | Leads | List, search, filter, and paginate Leads across their active and archived lifecycle states. |
| ADMIN-04 | Lead Details | Review and edit Lead information, notes, status, activity, and conversion into a Company. |
| ADMIN-05 | Companies | List, search, and filter Companies by lifecycle, Client access, invitation, and Pilot state. |
| ADMIN-06 | Company Details | Manage Company information, Client Workspace and invitation access, Pilot progress, the Company's Interview Requests, notes, and activity. |
| ADMIN-07 | Talent Applications | List, search, filter, and review Talent Pool applications by recruitment stage. |
| ADMIN-08 | Talent Application Details | Review candidate information, resume, assessments, notes, final-information requests and submissions, activity, and profile-creation readiness. |
| ADMIN-09 | Talent Profiles | List, search, filter, sort, and paginate active, visible, hidden, and archived Talent Profiles. |
| ADMIN-10 | Talent Profile Details | Review all professional and Admin-controlled information, files, visibility, account access, linked application, and profile activity. |
| ADMIN-11 | Create or Edit Talent Profile | Create a profile from an approved completed application or edit an existing profile, including Admin-controlled fields and a client-visible preview. |

Interview Requests are not a standalone Admin module. Each Company's requests
are managed inside Company Details so the Client, Company, Pilot, Talent, notes,
and scheduling context remain together.

Dashboard Interview Request items that require attention link directly to the
relevant Company's Interview Requests section.

Client invitation actions and Pilot management are handled inside Company
Details. Assessments, profile-completion requests, final-information review,
and profile creation begin inside Talent Application Details.

## 8. Admin Managed Content

| ID | Screen | Purpose |
|---|---|---|
| CMS-01 | Managed Content Home | Provide entry to each structured website-content area supported in V1. |
| CMS-02 | Trusted Partners | Manage trusted partner names, logo images, links, active state, and display order. |
| CMS-03 | Testimonials | Manage text and video testimonials, active state, order, and the primary testimonial. |
| CMS-04 | Services Hero Media | Manage and preview the Services hero video, cover image, and accessible media label. |
| CMS-05 | Case Studies | List and filter Draft and Published bilingual Case Studies. |
| CMS-06 | Case Study Editor | Create or edit English and German content, shared media, metadata, preview, and publication state. |
| CMS-07 | Insights | List and filter Draft and Published bilingual Insights. |
| CMS-08 | Insight Editor | Create or edit English and German content, shared metadata, preview, and publication state. |
| CMS-09 | Careers Roles | List and manage active, inactive, and featured internal BlihOps roles. |
| CMS-10 | Careers Role Editor | Create or edit an English-only internal BlihOps role and its email-application information. |
| CMS-11 | Pilot FAQs | Manage bilingual Pilot FAQs, active state, and display order. |

Simple content creation and editing may use a modal or drawer within its parent
screen. Case Studies, Insights, and Careers Roles use dedicated editors because
their content is more substantial.

## 9. Admin Settings

| ID | Screen | Purpose |
|---|---|---|
| SETTINGS-01 | Email Templates | List the Client invitation, profile-completion request, Talent invitation, password-reset, and Interview Request templates. |
| SETTINGS-02 | Email Template Editor | Edit a template's subject and body, validate supported variables, and preview the result. |
| SETTINGS-03 | Calendly Settings | Review connection status, event mapping, webhook configuration, last webhook received, and safe diagnostics. |

## 10. Shared Design Requirements

Every foundation screen must account for its relevant:

- Loading state
- Empty state
- Validation errors
- Retryable server errors
- Success feedback
- Upload progress and upload errors
- Confirmation dialogs
- Invalid or unavailable access
- Keyboard and screen-reader behavior
- Mobile and desktop layouts

These are design variants, not separate foundation screens in this inventory.
