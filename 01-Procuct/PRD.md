# BlihOps V1 — Product Requirements Document

## 1. Product Overview

### 1.1 Purpose

BlihOps connects global companies with pre-vetted Ethiopian software engineers.

Version 1 validates the outsourcing model by:

- Generating and qualifying company leads.
- Recruiting, assessing, and approving talent.
- Manually creating controlled Talent Profiles.
- Giving one invited representative per company access to a private Client Workspace.
- Letting that Client organize visible talent into planning-only Pods.
- Giving approved talent invitation-only access to maintain permitted profile information.
- Managing selected website content through structured admin tools.

### 1.2 Product Principles

- Admin operations are the product’s operational center.
- Access is invitation-only for protected client and talent experiences.
- Talent self-service begins only after admin approval and profile creation.
- Clients never contact talent directly through the platform.
- Website content management is bounded to defined content types.
- V1 favors clear manual control over workflow automation.

---

## 2. Products and Applications

### 2.1 BlihOps Web

The public website and protected client/talent entry point.

Includes:

- Marketing pages
- Contact and pilot-request forms
- Calendly booking
- Join Talent Pool
- Case Studies and Insights
- Careers roles
- Client Workspace
- Talent Portal

### 2.2 BlihOps Admin

The internal application for:

- Dashboard and operational activity
- Leads
- Companies and client access
- Talent Applications
- Talent Profiles
- Interview and pilot oversight
- Managed website content
- Email-template and Calendly settings

### 2.3 BlihOps API

The backend provides:

- Authentication and role-based authorization
- Invitations and token validation
- Lead, company, and client-workspace operations
- Talent Pod planning operations
- Talent recruitment and profile operations
- Content delivery by locale
- Notifications, file handling, and audit history

---

## 3. User Roles and Authorization

### 3.1 Roles

| Role | Description | Protected Application |
|------|-------------|-----------------------|
| Visitor | Unauthenticated public website user | None |
| Admin | Internal BlihOps operator | Admin Portal |
| Client | The invited representative of one approved company | Client Workspace |
| Talent | Approved engineer with an admin-created Talent Profile | Talent Portal |

System authorization roles:

- `ADMIN`
- `CLIENT`
- `TALENT`

### 3.2 Global Authorization Rules

- There is no public registration.
- Admin accounts are provisioned internally.
- Client and Talent accounts are invitation-only.
- Each company has one Client account and one Client Workspace in V1.
- A Client can access only their company’s workspace.
- A Talent can access only their own Talent Profile.
- A Talent account can be invited only after the Talent Profile exists.
- Unauthorized cross-application access redirects to the appropriate login or access-denied surface.
- The Admin Portal is not indexed by search engines.

---

## 4. Authentication and Invitations

### 4.1 Shared Authentication

Admin, Client, and Talent users can:

- Log in.
- Log out.
- Request a password-reset email.
- Set a new password using a valid reset link.

The system redirects authenticated users to their permitted application:

- Admin → Admin Dashboard
- Client → Client Workspace Dashboard
- Talent → Talent Profile Management

### 4.2 Client Account Invitation

An admin sends an invitation from an existing Company.

Requirements:

- The Company and its Client Workspace must exist.
- Only one active or pending Client account is allowed per company.
- The invitation is single-use and expires.
- A valid invitation lets the Client create a password and activate the account.
- An invalid, expired, or consumed invitation cannot activate access.
- Admin can resend or replace an expired pending invitation.

### 4.3 Talent Profile-Completion Token

After approving a Talent Application, an admin may send a secure profile-completion request.

This request:

- Uses a single-use, expiring token.
- Does not create a user account.
- Does not create a Talent Profile automatically.
- Opens a public token-protected final-information form.
- Becomes invalid after successful submission or expiry.

The form collects:

- Profile photo
- Professional headline
- Short bio
- Primary role
- Tech stack and secondary skills
- Years of experience
- Portfolio
- GitHub and LinkedIn
- Availability
- Earliest start date
- Preferred engagement

The resume submitted with the original Talent Application remains attached to the application and is not requested again during profile completion.

The submitted information returns to the associated Talent Application for admin review.

### 4.4 Talent Account Invitation

After reviewing the completed information and manually creating the Talent Profile, an admin may send a Talent account invitation.

Requirements:

- The Talent Application must be approved.
- A Talent Profile must already exist.
- The invitation is single-use and expires.
- The Talent creates a password and activates the account.
- The activated account opens Talent Profile Management, not a dashboard.
- Admin can resend or replace an expired pending invitation.

### 4.5 Password Reset

- Reset links are single-use and expire.
- Requesting a reset does not reveal whether an email exists.
- Successful password reset invalidates previous reset links.

---

## 5. Public Website

### 5.1 Public Pages

The website may include:

- Home
- Services
- How We Work
- About
- Case Studies
- Insights
- Careers
- Pilot
- Contact
- Join Talent Pool
- Client and Talent login entry
- Privacy and Terms

Only published content is visible publicly.

### 5.2 Contact Form

Purpose: collect general business inquiries.

Requirements:

- Validate required contact and company information.
- Capture the source page.
- Create a Lead with type `CONTACT`.
- Show a success state after submission.
- Prevent duplicate form submission during processing.

### 5.3 Pilot Request

Purpose: collect qualified pilot leads.

Requirements:

- Validate required company, contact, and pilot information.
- Capture the source page.
- Create a Lead with type `PILOT`.
- Show a success state with a Book Discovery Call action.

### 5.4 Book a Call

- Calendly provides scheduling.
- A successful Calendly webhook creates or updates a Lead with type `CALENDLY`.
- Duplicate webhook delivery must not create duplicate leads.

### 5.5 Join Talent Pool

Requirements:

- Collect the candidate’s application information.
- Require a resume.
- Accept supported file formats and size limits.
- Create one Talent Application.
- Show a confirmation state after successful submission.

### 5.6 Published Content

- Visitors can browse published Case Studies and Insights and open detail pages.
- Visitors can browse active Careers roles.
- Pilot pages display active FAQs for the selected locale.
- The website requests English or German content according to the active locale.

---

## 6. Admin Portal

### 6.1 Dashboard

The Admin Dashboard displays:

- New and active Leads
- Pilot Requests and Calendly bookings
- Talent Applications by current stage
- Pending profile-completion submissions
- Talent Profiles by visibility and availability
- Companies and pending Client invitations
- Interview Requests requiring attention
- Recent activity

### 6.2 Leads

Lead types:

- Contact
- Pilot
- Calendly
- Manually created

Lead statuses:

- New
- Contacted
- Discovery Scheduled
- In Discussion
- Qualified
- Converted
- Closed Lost
- Archived

Admin can:

- View, search, filter, and paginate Leads.
- Create a Lead manually.
- View and edit Lead details.
- Add internal notes.
- Change status.
- Archive and restore a Lead.
- Convert a qualified Lead into a Company.

Business rules:

- Every public Contact, Pilot Request, and completed Calendly booking creates or resolves to a Lead.
- A Lead never becomes a Company automatically.
- Conversion creates a Company and its Client Workspace, then marks the Lead as converted.
- Duplicate-company detection blocks accidental duplicate conversion.

### 6.3 Companies

Company statuses:

- Invitation Pending
- On Pilot
- Active
- Archived

Admin can:

- Create a Company manually or from a qualified Lead.
- View and edit company information.
- View the Client Workspace status, Client invitation status, Pilot status, Interview Requests, notes, and activity.
- Send or resend the single Client account invitation.
- Deactivate or reactivate the Client account.
- Update Pilot status and milestones.
- Archive and restore a Company.

Business rules:

- Creating a Company creates one Client Workspace.
- A company has at most one Client account in V1.
- Archiving a Company disables Client Workspace access without deleting history.
- Restoring a Company does not automatically reactivate a deactivated Client account.

### 6.4 Talent Applications

Recruitment stages:

- New
- Under Review
- Screening
- Technical Assessment
- English Assessment
- Remote Readiness Assessment
- Approved
- Profile Completion Requested
- Profile Information Submitted
- Profile Created
- Rejected
- Archived

Admin can:

- View, search, filter, and paginate applications.
- Review candidate details and uploaded files.
- Add internal notes.
- Record stage-specific outcomes.
- Advance or return an application to a valid stage.
- Reject an application at any stage with a reason.
- Approve a candidate after required assessments.
- Send or resend the profile-completion token.
- Review the final-information submission.
- Manually create a Talent Profile.
- Send or resend the Talent account invitation after profile creation.
- Archive and restore an application.

Business rules:

- One Talent Application represents one candidate submission.
- Only an approved application can receive a profile-completion request.
- Profile completion does not create a Talent Profile automatically.
- One Talent Profile may be created per approved application.
- A Talent account invitation is unavailable until the Talent Profile exists.

### 6.5 Talent Profiles

Admin can:

- View, search, filter, sort, and paginate profiles.
- Create a profile from an approved, completed application.
- View and edit all profile fields.
- Publish or hide a profile from clients.
- Archive and restore a profile.
- Review profile activity and its linked application.
- Send or resend the Talent account invitation.

Admin-managed fields include:

- Identity and contact information
- Seniority
- Hourly and monthly rates
- English level
- Verification state
- Assessment summary
- Internal notes
- Client visibility
- Lifecycle status

Talent-editable fields include:

- Profile photo
- Professional headline
- Short bio
- Primary role
- Tech stack and secondary skills
- Years of experience
- Portfolio, GitHub, and LinkedIn
- Resume
- Availability
- Earliest start date
- Preferred engagement

Business rules:

- Talent edits to permitted fields publish immediately.
- Talent cannot edit admin-managed fields.
- Only active, visible profiles expose Talent Profile details in Client Workspaces.
- When a profile used in a Pod becomes hidden or archived, its Pod membership is
  represented only by an unavailable placeholder with no profile access.
- Archiving a profile removes its client visibility and disables Talent Portal access.
- Restoring a profile does not automatically make it client-visible.

### 6.6 Interview and Pilot Oversight

Admin can:

- View Interview Requests across companies.
- Update Interview Request status.
- Record scheduling information and internal notes.
- View the related Company and Talent Profile.
- Update company Pilot status and milestones.

Interview Request statuses:

- Pending
- Scheduled
- Completed
- Cancelled

---

## 7. Client Workspace

### 7.1 Dashboard

Displays:

- Welcome and company context
- Current Pilot status
- Available Talent summary
- Active Talent Pod summary
- Recent workspace activity
- Upcoming or recent Interview Requests

### 7.2 Talent Directory

Client can:

- Browse active, visible Talent Profiles.
- Search by relevant profile text.
- Filter by role, skills, experience, and availability.
- Sort and paginate results.
- Open a Talent Profile.
- Add a visible Talent Profile to one or more Pods.

Talent cards may display:

- Photo
- Name
- Headline
- Role and experience
- Tech stack
- English level
- Availability
- Client-visible rate information

### 7.3 Talent Profile

Displays only client-visible fields:

- Photo, name, headline, and bio
- Role, seniority, and experience
- Tech stack and skills
- Portfolio and resume
- Availability
- English level and verification
- Client-visible rates

Client can add the profile to a Pod or request an interview, but cannot edit the
profile or contact the Talent directly.

### 7.4 Talent Pods

Purpose: let the Client organize visible Talent into private planning groups
without creating an assignment, reservation, or employment commitment.

Client can:

- View active Pods in their own Client Workspace.
- Create a Pod with a required name and optional planning note or objective.
- Rename or update the planning note.
- Add active, visible Talent Profiles from the Pod, Talent Directory, or Talent Profile.
- Remove Talent from a Pod.
- Optionally select one Pod Lead from the current Pod members.
- Clear or change the Pod Lead.
- Archive a Pod after confirmation.

Rules:

- A Pod belongs to exactly one Client Workspace.
- A Talent Profile may appear in more than one Pod.
- Duplicate membership within the same Pod is prevented.
- The Pod Lead is optional and must be a current member of that Pod.
- Removing the selected Pod Lead clears the lead assignment.
- If a member becomes hidden or archived, the Pod marks that member unavailable
  and blocks access to the Talent Profile.
- Pods are planning-only. Membership does not reserve, assign, hire, notify, or
  contact Talent and does not create an Interview Request or Pilot assignment.
- Clients can access and manage only Pods belonging to their own Workspace.

### 7.5 Interview Requests

Client can:

- Submit one Interview Request from a Talent Profile.
- Add relevant context for BlihOps.
- View their company’s requests.
- Track Pending, Scheduled, Completed, and Cancelled statuses.

All scheduling and direct coordination remains with BlihOps.

### 7.6 Pilot Status

Displays:

- Current Pilot
- Current status
- Timeline and milestones
- Assigned or participating talent when applicable
- Recent updates

### 7.7 Client Workspace Rules

- Only the company’s single active Client account can enter.
- The Client sees only their own Company data.
- Talent Pods are planning groups, not company-user teams or staffing assignments.
- There is no invitation flow for additional Client users.
- There is no direct messaging with Talent.

---

## 8. Talent Portal

### 8.1 Purpose

Provide an approved Talent with one focused, authenticated page for viewing and maintaining permitted fields on their admin-created profile.

### 8.2 Profile Management

Talent can:

- View their own profile.
- Edit permitted professional fields.
- Upload or replace their profile photo.
- Upload or replace their resume.
- Update portfolio and social links.
- Update availability and start information.
- Save changes and receive immediate success or validation feedback.

### 8.3 Talent Portal Rules

- There is no Talent Dashboard in V1.
- A Talent can access only their own profile.
- A profile must exist before account activation.
- Permitted changes are saved directly and reflected immediately.
- Admin-only fields are visible only when explicitly useful and are always read-only to Talent.
- Archived profiles cannot be edited through the Talent Portal.

---

## 9. Managed Website Content

### 9.1 Scope

The Admin Portal manages specific structured content. It does not provide unrestricted page editing.

### 9.2 Home — Trusted Logos

Admin can:

- Create, edit, activate/deactivate, reorder, and delete trusted-logo entries.

Fields:

- Organization name
- Logo image
- Optional website URL
- Active state
- Display order

Only active entries are public.

### 9.3 Home — Testimonials

Admin can:

- Create text or video testimonials.
- Edit, activate/deactivate, reorder, and delete testimonials.
- Select exactly one active testimonial as the primary testimonial for the managed-outsourcing section.

Shared fields:

- Person name
- Role or organization
- Type: text or video
- Active state
- Display order
- Primary selection

Text testimonial fields:

- Testimonial text
- Optional avatar

Video testimonial fields:

- Quote
- Video
- Optional cover image

If the primary testimonial becomes inactive or is deleted, the system requires another active primary selection before publishing the Home content state.

### 9.4 Services — Hero Media

The Services hero uses one global media object:

- Video
- Cover image
- Accessible media label or alt text
- Last updated information

Admin can replace the video or cover image and save the singleton object.

### 9.5 Case Studies

Each Case Study is one record with:

- English content tab
- German content tab
- Shared media and metadata
- Draft or Published status
- Featured state

Localized fields:

- Title
- Slug
- Summary
- Body content
- Services and outcomes copy

Shared fields:

- Client
- Category
- Hero image
- Tags
- Featured state

Rules:

- English and German required fields must both validate before publication.
- Drafts may be incomplete.
- Publication and unpublication apply to both locales together.
- Slugs must be unique within their locale.

### 9.6 Insights

Each Insight is one record with:

- English content tab
- German content tab
- Shared metadata
- Draft or Published status
- Featured state

Localized fields:

- Title
- Slug
- Excerpt
- Body content

Shared fields:

- Author
- Category
- Tags
- Hero image
- Read time
- Featured state

Rules:

- English and German required fields must both validate before publication.
- Drafts may be incomplete.
- Publication and unpublication apply to both locales together.
- Slugs must be unique within their locale.

### 9.7 Careers Roles

Careers roles are English-only.

Fields:

- Title
- Department
- Location
- Employment type
- Summary and overview
- Responsibilities
- Requirements
- Active state
- Featured state

Admin can create, edit, activate/deactivate, feature/unfeature, and delete a role. Only active roles appear publicly.

### 9.8 Pilot FAQs

Each FAQ entry contains:

- English question and answer
- German question and answer
- Active state
- Display order

Admin can create, edit, activate/deactivate, reorder, and delete FAQs.

Rules:

- Both locales are required before an FAQ can become active.
- Only active FAQs appear on the Pilot page for the selected locale.

### 9.9 Content Safety Rules

- Destructive deletion requires confirmation.
- Upload failures preserve unsaved form data where possible.
- Published or active content changes create activity records.
- Public content delivery selects content using the requested locale.
- Missing bilingual content cannot be published with an English fallback.

---

## 10. Settings

### 10.1 Email Templates

Admin can view and update templates for:

- Client account invitation
- Talent profile-completion request
- Talent account invitation
- Password reset
- Interview Request notification

Template variables must be validated before saving.

### 10.2 Calendly

Admin can view:

- Connection status
- Webhook configuration
- Event mapping
- Last webhook received

Sensitive credentials are never displayed in full.

---

## 11. Notifications

### 11.1 Admin Notifications

- New Contact or Pilot Request
- New Calendly booking
- New Talent Application
- Talent profile information submitted
- Client invitation accepted
- Talent account invitation accepted
- New Interview Request

### 11.2 Client Notifications

- Client account invitation
- Password reset
- Interview Request status update
- Pilot status update
- A Talent Profile used in a Pod becoming unavailable

### 11.3 Talent Notifications

- Profile-completion request
- Talent account invitation
- Password reset
- Important profile-access change

---

## 12. File Management

- Talent Applications require a resume.
- Profile completion may include a profile photo; it reuses the resume from the original Talent Application.
- Talent Portal supports profile photo and resume replacement.
- Managed content supports the image and video types required by each content model.
- Files must pass configured type and size validation.
- Private candidate and talent files require authorization.
- Replaced or deleted files follow the platform’s storage cleanup policy.

---

## 13. Activity and Audit

The system records important actions with actor, timestamp, resource, and action.

At minimum:

- Lead creation, status changes, conversion, archival, and restoration
- Company creation, invitation activity, access changes, and archival
- Talent Application stage changes, approval, rejection, and token activity
- Talent Profile creation, field updates, visibility changes, and archival
- Client and Talent account activation
- Talent Pod creation, detail changes, membership changes, Pod Lead changes, and archival
- Interview Request creation and status changes
- Pilot status changes
- Managed-content creation, publication, activation, ordering, and deletion

Talent-originated profile changes must be distinguishable from Admin changes.

---

## 14. Non-Functional Requirements

### Security

- Role and ownership checks are enforced server-side.
- Tokens and reset links are single-use and expire.
- Private files use protected access.
- Sensitive credentials and internal fields are never exposed to unauthorized users.

### Accessibility

- Core workflows are keyboard accessible.
- Forms provide programmatic labels and actionable validation messages.
- Status is not communicated by color alone.
- Upload controls expose progress and errors accessibly.

### Responsive Behavior

- Public pages, Client Workspace, Talent Portal, and core Admin workflows support desktop and mobile layouts.
- Data-heavy Admin tables may use responsive alternatives without hiding required actions.

### Reliability

- Forms prevent accidental duplicate submission.
- Failed mutations show retryable errors without falsely reporting success.
- Locale-aware content requests return only published content.

---

## 15. V1 Assumptions

- One seeded or internally provisioned Admin account is sufficient initially.
- Each company has one Client account and one Client Workspace.
- Talent Pods are private planning aids and never imply reservation or assignment.
- Each approved Talent Profile has at most one Talent account.
- Recruitment communication and interview scheduling remain operationally managed by BlihOps.
- Calendly handles call scheduling.
- Talent self-service is limited to the explicitly permitted profile fields.
- The backend will store and return locale-specific content according to the requested locale.

---

## 16. Out of Scope for Version 1

### Products

- BlihOps Skills
- Public Talent Marketplace

### Accounts and Security

- Public registration
- Multiple Client accounts per company
- Internal role-management UI
- Multi-factor authentication

### Client Features

- Company-user team management
- Staffing, reservation, or assignment actions through Talent Pods
- Direct messaging with Talent
- Contracts
- Payments and invoicing
- Advanced analytics

### Talent Features

- Talent-created profiles before Admin approval
- Editing rates, assessments, verification, visibility, status, or internal notes
- Talent Dashboard
- Client discovery or marketplace browsing

### Operations

- AI talent matching
- Automated hiring decisions
- Fully automated recruitment communication
- General-purpose page-builder CMS
- Advanced reporting and workflow automation
