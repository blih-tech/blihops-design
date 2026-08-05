# BlihOps V1 User Personas

## 1. Purpose

This document defines the people whose goals, responsibilities, constraints,
and journeys guide the BlihOps V1 experience.

These are role-and-journey personas. They avoid invented names, demographics,
and biographies and instead focus on what each person is trying to accomplish,
where they enter the BlihOps lifecycle, what they need, and where their access
ends.

The protected product roles remain Admin, Client, and Talent. Prospective
Clients and Talent Pool Candidates use the public website as unauthenticated
Visitors.

## 2. Persona Set and Relationships

### Company journey

    Prospective Client
        -> submits a Contact or Pilot Request, or books a call
        -> becomes or updates a Lead
        -> discusses the engagement with BlihOps manually
        -> Admin creates the Company and Client Workspace after agreement
        -> the Company selects one Approved Client Representative
        -> the representative receives invitation-only Workspace access

The Approved Client Representative may be the original contact or a different
person selected by the Company. V1 permits only one Client account per Company.

### Talent journey

    Talent Pool Candidate
        -> submits a Talent Pool application
        -> completes Admin-managed review and assessments
        -> receives approval
        -> receives and submits a mandatory profile completion request
        -> Admin reviews the combined application information
        -> Admin creates the Talent Profile and account
        -> receives an account invitation
        -> activates the account and maintains permitted profile information

The Talent Pool application collects the information needed for Admin review in
Round 1. After approval, the candidate must provide the remaining profile-building
information through a mandatory Round 2 completion request. Admin creates the
Profile and account only after reviewing the combined information, then sends the
account invitation.
Careers applicants are not part of this V1 persona set.

## 3. Primary Persona - BlihOps Admin

### Context

The Admin is one general BlihOps operator for V1. The Admin handles sales,
client onboarding, talent recruitment, Talent Profile control, interview and
Pilot coordination, selected website content, and operational settings.

### Primary goals

- Process and qualify new Leads efficiently.
- Move agreed companies into a controlled onboarding journey.
- Maintain one Client representative and Workspace per Company.
- Recruit and assess Talent consistently.
- Create accurate, client-safe Talent Profiles from completed Round 1 and Round 2
  application information.
- Coordinate Interview Requests and Pilots through BlihOps.
- Keep selected public content accurate and current.
- Preserve reliable operational history.

### Core tasks

- Review Contact, Pilot, Calendly, and manually created Leads.
- Qualify, close, archive, or convert Leads into Companies.
- Create Companies and Client Workspaces after manual agreement.
- Invite and manage the single Client representative.
- Review Talent Pool applications and record assessments.
- Approve or reject candidates.
- Request mandatory Round 2 information from approved candidates.
- Create Talent Profiles and accounts from approved applications only after Round 2
  is submitted.
- Control Talent Profiles and invitation status.
- Invite Approved Talent after their profile exists.
- Update Interview Request and Pilot information.
- Manage the structured website content defined for V1.
- Review activity, notifications, email templates, and Calendly information.

### Needs and expectations

- Clear pipeline and lifecycle states.
- Fast lists, filters, and contextual actions.
- Visible invitation, delivery, and account states.
- Strong separation between internal and client-visible information.
- Clear warnings before destructive or access-changing actions.
- Predictable links between related resources.
- Truthful success and failure feedback without lost work.

### Frustrations and risks

- Duplicate Leads, Companies, applications, or profiles.
- Confusing approval, account creation, and account activation.
- Accidentally exposing private or internal information.
- Losing track of invitations, submissions, Interview Requests, or Pilot work.
- Invalid or unclear lifecycle transitions.
- Publishing incomplete bilingual content.

### Access and V1 boundaries

- Admin has operational control over all V1 resources.
- Admin accounts are provisioned internally.
- V1 has one general Admin persona rather than specialized internal roles.
- BlihOps retains manual control over recruitment, matching, interviews, and
  client onboarding.

## 4. Persona - Prospective Client

### Context

The Prospective Client represents a company exploring BlihOps. They may be a
decision-maker, hiring stakeholder, technical leader, or another company
contact. They are not authenticated and may not become the final Approved
Client Representative.

### Primary goals

- Understand the BlihOps service and delivery model.
- Evaluate BlihOps credibility and relevant outcomes.
- Explain the Company's engineering or Pilot needs.
- Contact BlihOps or schedule a discussion with minimal friction.
- Understand what happens after submitting an inquiry.

### Core tasks

- Browse public marketing content, Case Studies, and Insights.
- Submit a Contact form or Pilot Request.
- Book a call through Calendly.
- Continue qualification and agreement discussions manually with BlihOps.

### Needs and expectations

- Clear positioning, services, process, and trust signals.
- Simple forms that request only relevant information.
- Explicit confirmation after submitting an inquiry.
- Clear expectations that BlihOps will follow up manually.
- No implication that a public form creates a Company or Client account.

### Frustrations and risks

- Unclear engagement steps or Pilot expectations.
- Repetitive or excessive form fields.
- No confirmation after submission.
- Being pushed into account creation before an agreement exists.

### Journey boundary

- Public actions create or update a Lead for Admin review.
- A Lead never becomes a Company automatically.
- Company and Workspace creation happen only after manual agreement.
- The Company may nominate a different person as its Client representative.

## 5. Persona - Approved Client Representative

### Context

The Approved Client Representative is the one person selected by an agreed
Company to use its private Client Workspace. They may or may not be the person
who originally contacted BlihOps.

### Primary goals

- Activate access without confusion.
- Understand the current Company and Pilot context.
- Find relevant, available engineers.
- Review clear, client-safe Talent information.
- Organize potential Talent into private planning Pods.
- Request interviews through BlihOps.
- Track Interview Requests and Pilot progress.

### Core tasks

- Accept the Client invitation and create a password.
- Enter the Company's Client Workspace.
- Browse, search, filter, sort, and paginate visible Talent Profiles.
- Review permitted Talent details and authorized files.
- Create and manage planning-only Talent Pods.
- Submit an Interview Request with relevant context.
- Review Interview Request and Pilot updates.

### Needs and expectations

- Clear Company identity and Workspace orientation.
- A focused interface without internal BlihOps complexity.
- Accurate professional, availability, verification, and approved
  client-visible commercial information.
- Clear feedback after requests and Pod changes.
- Confidence that BlihOps coordinates the next step.
- Clear explanations when a Profile or Workspace becomes unavailable.

### Frustrations and risks

- Seeing private or internal Talent information.
- Unclear Talent availability or commercial information.
- Confusing a planning Pod with staffing or assignment.
- Uncertainty after submitting an Interview Request.
- Losing Workspace access without an explanation.

### Access and V1 boundaries

- One Client account is allowed per Company.
- The Client can access only their own Company's Workspace and data.
- Talent Pods are private planning groups only.
- Pod membership does not reserve, assign, hire, notify, or contact Talent.
- The Client cannot edit Talent Profiles or contact Talent directly.
- The Client cannot invite additional users or manage company-user teams.
- Contracts, payments, billing, and direct messaging are outside V1.

## 6. Persona - Talent Pool Candidate

### Context

The Talent Pool Candidate is an engineer applying specifically to the BlihOps
Talent Pool. This persona does not represent a Careers applicant. The Candidate
remains unauthenticated throughout application, recruitment, and account
invitation.

### Primary goals

- Submit a complete Talent Pool application successfully.
- Understand the recruitment process and current expectations.
- Complete required assessments and provide complete professional information.
- Know what happens after approval and account invitation.

### Core tasks

- Complete the Talent Pool application.
- Upload the required resume.
- Participate in BlihOps-managed screening and assessments.
- After approval, submit the mandatory profile completion form with the remaining
  information needed to create a client-safe Profile.

### Needs and expectations

- Clear application requirements and file constraints.
- Accessible forms that preserve valid work after recoverable errors.
- Honest confirmation after application submission.
- Clear separation between application approval, mandatory profile completion,
  Admin-created Profile/account, and account invitation.
- Clear invitation handling that does not expose candidate information.

### Frustrations and risks

- Confusing account creation with account activation.
- Losing form data after validation, upload, or server errors.
- Unclear next steps after submission.
- Receiving account-oriented language before a profile exists.

### Journey boundary

- Round 1 application submission creates a Talent Application containing the
  information needed for Admin review and assessment.
- Approval requires Admin to send a Round 2 profile completion request.
- Round 2 submission adds the remaining profile-building information to the
  application.
- Admin can create one Talent Profile and account only after reviewing the combined
  Round 1 and Round 2 information.
- The account invitation activates access after the account is created.
- The persona becomes Approved Talent once the Profile and account exist.

## 7. Persona - Approved Talent

### Context

Approved Talent is an engineer whose Talent Profile and account have been created
by Admin from an approved application. They receive an invitation to activate
the account and use a focused Profile Management experience.

### Primary goals

- Activate the Talent account successfully.
- Understand what profile information they may maintain.
- Keep professional information accurate and current.
- Replace their profile photo or resume safely.
- Update availability, start information, and professional links.
- Understand whether saved changes are live.

### Core tasks

- Accept the Talent account invitation and create a password.
- Open the single Profile Management destination.
- Update permitted professional fields.
- Upload or replace a profile photo and resume.
- Save changes and resolve validation or concurrent-update conflicts.

### Needs and expectations

- A focused profile experience with no unrelated dashboard.
- Clear editable and protected field boundaries.
- Accessible uploads with progress, validation, cancellation, and recovery.
- Immediate and truthful save feedback.
- Confidence that private and internal information remains protected.
- Clear guidance when profile access is unavailable.

### Frustrations and risks

- Confusing account activation with Admin account creation.
- Not knowing which fields can be changed.
- Overwriting a newer Admin update.
- Losing an existing file after a failed replacement.
- Uncertainty about when permitted changes become client-visible.

### Access and V1 boundaries

- Talent can access only their own Admin-created Talent Profile.
- Talent lands on Profile Management; there is no Talent Dashboard.
- Talent may edit only the professional fields permitted by the PRD.
- Talent cannot edit rates, assessments, verification, client visibility,
  lifecycle status, or internal notes.
- Permitted saved changes take effect immediately where the profile is visible.
- An archived profile blocks Talent Portal editing.

## 8. Cross-Persona UX Principles

- Preserve the distinction between public intent, operational approval,
  resource creation, and account access.
- Never imply that a public form automatically creates a Company, Workspace,
  Talent Profile, or account.
- Make invitation purpose, token purpose, expiry, and next steps explicit.
- Show only information and actions permitted for the current persona.
- Keep Admin workflows complete while keeping Client and Talent experiences
  focused.
- Preserve safe input during recoverable validation, upload, and save failures.
- Confirm successful actions truthfully and provide safe recovery paths.
- Use consistent lifecycle language across future User Flows and the Screen
  Inventory.

## 9. Explicit Persona Exclusions

The V1 persona set does not introduce:

- Careers applicants
- Public account holders
- Additional Client users or company-user teams
- Specialized Admin roles
- Talent who create their own initial profile
- Client-to-Talent direct communication
- Public Talent Marketplace users
- BlihOps Skills users
