# User Personas

## 1. Overview

These personas translate the V1 product roles into UX priorities. They guide workflows, navigation, screen hierarchy, content, and permission boundaries.

---

## 2. Primary Persona — Admin

### Description

Admins operate the BlihOps platform. They qualify company leads, onboard the single Client representative, run talent recruitment, create and control Talent Profiles, coordinate interviews and pilots, and manage selected website content.

### Primary Goals

- Process new leads quickly.
- Keep the sales pipeline accurate.
- Recruit and assess talent consistently.
- Collect final talent information without creating profiles prematurely.
- Create high-quality Talent Profiles.
- Onboard clients and talent with clear invitation states.
- Keep public content current.

### Needs

- Fast operational lists and filters
- Clear status and stage transitions
- Contextual actions on resource details
- Visible invitation and token state
- Strong separation between talent-editable and admin-only fields
- Reliable activity history
- Efficient bilingual content authoring

### Risks and Friction

- Confusing the profile-completion request with account activation
- Losing track of pending invitations
- Publishing incomplete bilingual content
- Accidentally exposing internal profile data
- Repetitive switching between related resources

---

## 3. Secondary Persona — Client

### Description

The Client is the single approved representative of a company. They use a private Client Workspace to review available engineers, request interviews through BlihOps, and monitor pilot progress.

### Primary Goals

- Find relevant available engineers.
- Compare clear Talent Profile information.
- Request an interview with minimal effort.
- Understand the status of interview requests.
- Track the current pilot.

### Needs

- Low-friction invitation activation
- Simple navigation
- Useful search and filters
- Consistent, client-safe profile information
- Clear status feedback
- Confidence that requests are handled by BlihOps

### Risks and Friction

- Too much internal or technical operational detail
- Unclear availability or rates
- Uncertainty after submitting an Interview Request
- Confusion about why direct talent contact is unavailable

### V1 Boundary

The Client can organize visible Talent into planning-only Pods and optionally
select a Pod Lead. The Client cannot invite additional users, create company-user
teams, turn Pod membership into a staffing assignment, edit Talent Profiles, or
contact Talent directly.

---

## 4. Secondary Persona — Talent

### Description

Talent are approved engineers. They first receive a tokenized request to submit final professional information. After an admin reviews that submission and manually creates a Talent Profile, they receive a separate account invitation to maintain permitted profile information.

### Primary Goals

- Complete the final-information request successfully.
- Activate the Talent account when invited.
- Keep professional information accurate.
- Replace a resume or profile photo.
- Keep skills, links, and availability current.

### Needs

- Clear distinction between profile completion and account activation
- A focused Profile Management page
- Understandable field permissions
- Reliable uploads
- Immediate save and validation feedback
- Confidence that private and internal data remains protected

### Risks and Friction

- Assuming the first token creates an account
- Not understanding why certain fields are read-only
- Losing form work after an upload or save error
- Uncertainty about whether changes are live

### V1 Boundary

Talent cannot create their initial profile, access a dashboard, or edit rates, assessments, verification, visibility, status, or internal notes.

---

## 5. Supporting Persona — Visitor

### Description

Visitors evaluate BlihOps through the public website. They may become company leads, Talent applicants, or Careers candidates.

### Primary Goals

- Understand BlihOps services and credibility.
- Review Case Studies and Insights in the selected locale.
- Request a pilot or contact BlihOps.
- Book a call.
- Apply to the Talent Pool.
- Review active Careers roles.

### Needs

- Clear positioning and trust signals
- Fast, accessible forms
- Explicit submission confirmation
- Correct English or German content
- Predictable routes into protected applications

---

## 6. Shared UX Principles

- Clarity over customization
- Speed over unnecessary steps
- Immediate and truthful feedback
- Permission-aware interfaces
- Consistent lifecycle terminology
- Accessible forms and status communication
- Preserved user input during recoverable errors
- No hidden automatic transitions between business stages

---

## 7. Design Priority

The Admin experience is the operational priority. Client and Talent experiences remain deliberately focused:

- Client: discover talent, request interviews, track the pilot.
- Talent: complete requested information, then maintain permitted profile fields.

Neither experience should inherit Admin complexity.
