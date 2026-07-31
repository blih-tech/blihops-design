# BlihOps V1 — User Flows

## 1. Overview

This document is the single source of truth for BlihOps V1 user flows. It derives from the Product Brief, PRD, Information Architecture, Navigation, and Personas.

Each flow defines:

- Actor
- Trigger
- Preconditions
- Main flow
- Meaningful alternatives and errors
- Postconditions

---

## 2. Shared Flow Conventions

### 2.1 Roles

- Visitor
- Admin
- Client
- Talent

### 2.2 Feedback

- Forms show field-level validation beside the affected field.
- Successful mutations show an explicit success state or confirmation.
- Failed mutations preserve user input where safe and provide a retry.
- Destructive actions require confirmation.
- Loading states prevent duplicate submission.

### 2.3 Access

- Protected routes validate authentication and authorization before loading private data.
- Client access is limited to one Company and its Client Workspace.
- Talent access is limited to the Talent’s own admin-created profile.
- Invalid or expired tokens never reveal protected form data.

---

# 3. Public Website

## Flow 3.1 — Browse Public Website and Select Locale

**Actor:** Visitor  
**Trigger:** The Visitor opens any public page.  
**Preconditions:** The requested page is public.

### Main Flow

1. The system loads the public website shell.
2. The Visitor navigates among marketing pages, Case Studies, Insights, Careers, and Pilot content.
3. The Visitor selects English or German.
4. The system preserves the selected locale.
5. Locale-aware content is requested for Case Studies, Insights, and Pilot FAQs.
6. Only published or active content is displayed.

### Alternatives and Errors

- If a public resource does not exist, the system shows a not-found page.
- If published content cannot load, the page shows a retryable error without exposing draft content.
- If no published content exists for a listing, an intentional empty state is shown.

### Postconditions

- The Visitor remains unauthenticated.
- The selected locale is used for subsequent locale-aware content.

---

## Flow 3.2 — Submit Contact Form

**Actor:** Visitor  
**Trigger:** The Visitor submits the Contact form.  
**Preconditions:** The Contact form is available.

### Main Flow

1. The Visitor enters the required contact and company information.
2. The page records the source page.
3. The Visitor submits the form.
4. The system disables repeated submission while processing.
5. The system validates the input.
6. The system creates a Lead with type `CONTACT` and status `NEW`.
7. The Admin notification is created.
8. The Visitor sees a success state.

### Alternatives and Errors

- Invalid fields are identified and the Visitor remains on the form.
- A duplicate submission returns the original successful outcome rather than creating a second Lead.
- A server failure preserves the form and offers retry.

### Postconditions

- One Contact Lead exists.
- The Lead appears in the Admin Portal.

---

## Flow 3.3 — Request a Pilot

**Actor:** Visitor  
**Trigger:** The Visitor submits the Pilot Request form.  
**Preconditions:** The Pilot Request form is available.

### Main Flow

1. The Visitor enters the required contact, company, and pilot information.
2. The page records the source page.
3. The Visitor submits the form.
4. The system validates the input.
5. The system creates a Lead with type `PILOT` and status `NEW`.
6. The Admin notification is created.
7. The success state offers a Book Discovery Call action.
8. The Visitor may open Calendly.

### Alternatives and Errors

- Invalid fields are identified without clearing valid values.
- Duplicate submission does not create another Lead.
- If creation fails, the form remains available with a retry.

### Postconditions

- One Pilot Lead exists.
- The Visitor has a clear next step.

---

## Flow 3.4 — Book a Call

**Actor:** Visitor  
**Trigger:** The Visitor selects Book a Call.  
**Preconditions:** Calendly is configured.

### Main Flow

1. Calendly opens with the configured event.
2. The Visitor chooses a time and completes the booking.
3. Calendly sends a webhook.
4. The system verifies and processes the webhook.
5. A Lead with type `CALENDLY` is created or an existing matching Lead is updated.
6. The booking activity is recorded.

### Alternatives and Errors

- Cancelling Calendly creates no booking activity.
- Duplicate webhook delivery is processed idempotently.
- Invalid webhooks are rejected and recorded for Admin diagnosis.

### Postconditions

- One Lead represents the completed booking.

---

## Flow 3.5 — Submit Talent Application

**Actor:** Visitor  
**Trigger:** The Visitor submits Join Talent Pool.  
**Preconditions:** The application form is open.

### Main Flow

1. The candidate enters the required personal and professional information.
2. The candidate uploads a resume.
3. The candidate submits the application.
4. The system validates fields, file type, and file size.
5. The system creates a Talent Application with status `NEW`.
6. The system creates an Admin notification.
7. The candidate sees a confirmation state.

### Alternatives and Errors

- Upload or validation errors identify the affected input.
- A failed save preserves the entered information where safe.
- Duplicate submission is detected and does not create another application.

### Postconditions

- One Talent Application exists in the recruitment pipeline.
- No Talent Profile or account exists yet.

---

## Flow 3.6 — Browse Careers Roles

**Actor:** Visitor  
**Trigger:** The Visitor opens Careers.  
**Preconditions:** The Careers page is public.

### Main Flow

1. The system loads active English-only roles.
2. Featured roles receive the defined emphasis.
3. The Visitor browses role information.
4. The Visitor opens a role detail when available.

### Alternatives and Errors

- If no roles are active, the page shows a no-open-roles state.
- If a role becomes inactive, its public detail is no longer available.

### Postconditions

- Only active Careers roles are visible.

---

# 4. Authentication, Invitations, and Access

## Flow 4.1 — Admin Login

**Actor:** Admin  
**Trigger:** The Admin opens the Admin login page.  
**Preconditions:** An Admin account exists and no valid session is active.

### Main Flow

1. The Admin enters email and password.
2. The Admin submits the form.
3. The system validates the credentials and `ADMIN` role.
4. The system creates an authenticated session.
5. The Admin is redirected to the Admin Dashboard.

### Alternatives and Errors

- A valid existing session redirects directly to the Dashboard.
- Invalid credentials show a generic authentication error.
- A non-Admin account is denied Admin Portal access.
- A server failure keeps the Admin on the login page with retry.

### Postconditions

- The Admin has an active Admin session.

---

## Flow 4.2 — Accept Client Account Invitation

**Actor:** Client  
**Trigger:** The Client opens an invitation link.  
**Preconditions:** The Company and Client Workspace exist, no other Client account is active or pending, and the token is valid.

### Main Flow

1. The system validates the invitation before displaying account fields.
2. The Client reviews the Company context.
3. The Client creates and confirms a password.
4. The system validates the password.
5. The system creates or activates the `CLIENT` account.
6. The invitation is consumed.
7. The Client is authenticated.
8. The Client Workspace Dashboard opens.

### Alternatives and Errors

- An expired or invalid token shows an unavailable-invitation state.
- An already consumed token offers Login.
- A replaced invitation rejects the older token.
- A failed activation preserves no password and allows safe retry.

### Postconditions

- The Company has one active Client account.
- The invitation cannot be reused.

---

## Flow 4.3 — Accept Talent Account Invitation

**Actor:** Talent  
**Trigger:** The Talent opens the account invitation sent after profile creation.  
**Preconditions:** The Talent Application is approved, the Talent Profile exists and is not archived, and the invitation is valid.

### Main Flow

1. The system validates the invitation.
2. The Talent reviews the account-activation context.
3. The Talent creates and confirms a password.
4. The system validates the password.
5. The system creates or activates the `TALENT` account linked to the existing profile.
6. The invitation is consumed.
7. The Talent is authenticated.
8. Talent Profile Management opens.

### Alternatives and Errors

- An expired or invalid token shows an unavailable-invitation state.
- A consumed token offers Login.
- A missing or archived Talent Profile blocks activation and instructs the Talent to contact BlihOps.
- A server failure allows retry without consuming the token.

### Postconditions

- One Talent account is linked to the existing Talent Profile.
- No dashboard is created or displayed.

---

## Flow 4.4 — Client or Talent Login

**Actor:** Client or Talent  
**Trigger:** The user opens Login.  
**Preconditions:** The invitation was accepted and the account is active.

### Main Flow

1. The user enters email and password.
2. The system validates credentials and role.
3. The system creates a session.
4. A Client opens the Client Workspace Dashboard.
5. A Talent opens Talent Profile Management.

### Alternatives and Errors

- A valid session bypasses Login.
- Invalid credentials show a generic authentication error.
- A deactivated Client account cannot enter the workspace.
- An archived Talent Profile blocks Talent Portal access.

### Postconditions

- The user enters only the application permitted by their role.

---

## Flow 4.5 — Reset Password

**Actor:** Admin, Client, or Talent  
**Trigger:** The user selects Forgot Password.  
**Preconditions:** The user is unauthenticated.

### Main Flow

1. The user enters an email address.
2. The system accepts the request without confirming account existence.
3. If an eligible account exists, the system sends a reset link.
4. The user opens the link.
5. The system validates the token.
6. The user creates and confirms a new password.
7. The system updates the password and consumes the token.
8. The user returns to Login.

### Alternatives and Errors

- An invalid or expired link offers a new reset request.
- A consumed link cannot be reused.
- Password validation errors remain on the reset form.

### Postconditions

- A successful reset invalidates previous reset links.

---

## Flow 4.6 — Log Out

**Actor:** Admin, Client, or Talent  
**Trigger:** The user selects Log Out.  
**Preconditions:** A session is active.

### Main Flow

1. The system invalidates the session.
2. Local session state is cleared.
3. The user is redirected to the appropriate Login page.

### Alternatives and Errors

- If server-side invalidation fails, local credentials are still cleared and protected data is removed from view.

### Postconditions

- Protected routes require authentication again.

---

## Flow 4.7 — Handle Unauthorized or Missing Resource

**Actor:** Admin, Client, or Talent  
**Trigger:** The user opens a protected resource.  
**Preconditions:** The route exists or was previously valid.

### Main Flow

1. The system checks session, role, ownership, and resource state.
2. An authorized user receives the resource.
3. An unauthenticated user is redirected to Login.
4. An authenticated but unauthorized user sees Access Denied.
5. A missing resource shows Not Found.

### Alternatives and Errors

- Archived resources follow role-specific behavior: Admin may view permitted archive detail; Client and Talent access is blocked.

### Postconditions

- No unauthorized private information is disclosed.

---

# 5. Lead Management

## Flow 5.1 — Create Lead Manually

**Actor:** Admin  
**Trigger:** The Admin selects Create Lead.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The Admin enters Lead and company information.
2. The system checks for a matching Company or Lead.
3. The Admin submits valid information.
4. The system creates a manually sourced Lead with status `NEW`.
5. The Lead detail opens.

### Alternatives and Errors

- A possible duplicate is shown before creation; the Admin may cancel or proceed when permitted.
- Validation or save errors preserve the form.

### Postconditions

- The Lead appears in the active pipeline.

---

## Flow 5.2 — Review and Update Lead

**Actor:** Admin  
**Trigger:** The Admin opens a Lead.  
**Preconditions:** The Lead exists.

### Main Flow

1. The system shows company information, contact information, source, status, notes, and activity.
2. The Admin edits permitted information or adds an internal note.
3. The Admin saves.
4. The system validates and records the changes.
5. The activity timeline identifies the actor and update.

### Alternatives and Errors

- A stale update prompts the Admin to refresh before overwriting newer information.
- Failed saves preserve the edit.

### Postconditions

- The Lead reflects the latest saved information.

---

## Flow 5.3 — Change Lead Status

**Actor:** Admin  
**Trigger:** The Admin selects a new Lead status.  
**Preconditions:** The Lead is active and the transition is allowed.

### Main Flow

1. The Admin selects the destination status.
2. The system validates the transition.
3. The system updates the Lead.
4. The pipeline and activity timeline update.

### Alternatives and Errors

- An invalid transition is rejected with a reason.
- Converting to `CONVERTED` is completed through the conversion flow, not a direct status edit.

### Postconditions

- The Lead appears in the correct pipeline view.

---

## Flow 5.4 — Convert Qualified Lead to Company

**Actor:** Admin  
**Trigger:** The Admin selects Convert to Company.  
**Preconditions:** The Lead is qualified and no matching Company exists.

### Main Flow

1. The conversion form is prefilled from Lead data.
2. The Admin reviews and completes Company information.
3. The system checks for duplicate Companies.
4. The Admin confirms conversion.
5. The system creates the Company.
6. The system creates one Client Workspace.
7. The Lead becomes `CONVERTED`.
8. The Company detail opens.
9. The Admin may send the single Client account invitation.

### Alternatives and Errors

- Duplicate detection blocks conversion and links the existing Company when permitted.
- The Admin may create the Company and invite the Client later.
- A transactional failure leaves the Lead unconverted and creates no partial Company/Workspace pair.

### Postconditions

- One Company and one Client Workspace exist.
- The source Lead links to the Company.

---

## Flow 5.5 — Archive or Restore Lead

**Actor:** Admin  
**Trigger:** The Admin selects Archive or Restore.  
**Preconditions:** The Lead exists.

### Main Flow

1. For Archive, the system requests confirmation.
2. The Admin confirms.
3. The Lead moves to Archived.
4. From Archived Leads, the Admin may select Restore.
5. The Lead returns to the appropriate active status or a defined restored status.

### Alternatives and Errors

- Cancelling confirmation changes nothing.
- A failed lifecycle change leaves the previous status intact.

### Postconditions

- The Lead is visible in the correct active or archived view.

---

# 6. Company and Client Access

## Flow 6.1 — Create Company Manually

**Actor:** Admin  
**Trigger:** The Admin selects Create Company.  
**Preconditions:** No matching Company exists.

### Main Flow

1. The Admin enters Company information.
2. The system validates and checks duplicates.
3. The Admin submits.
4. The system creates the Company and one Client Workspace.
5. The Company detail opens.

### Alternatives and Errors

- Duplicate detection blocks creation and links the existing Company.
- A transactional failure creates neither resource.

### Postconditions

- The Company and Client Workspace exist without an automatic Client invitation.

---

## Flow 6.2 — Review or Update Company

**Actor:** Admin  
**Trigger:** The Admin opens a Company.  
**Preconditions:** The Company exists.

### Main Flow

1. The system displays Company information, Workspace state, the single Client account or pending invitation, Pilot status, Interview Requests, notes, and activity.
2. The Admin edits permitted Company information or notes.
3. The system validates and saves changes.
4. Activity is recorded.

### Alternatives and Errors

- A failed save preserves the edit.
- Archived Companies are read-only except for permitted restoration and notes.

### Postconditions

- Company information reflects the latest saved state.

---

## Flow 6.3 — Send or Resend Client Invitation

**Actor:** Admin  
**Trigger:** The Admin selects Invite Client or Resend Invitation.  
**Preconditions:** The Company is not archived and has no active Client account.

### Main Flow

1. The Admin enters or confirms the Client email.
2. The system verifies that no active or pending conflicting Client account exists.
3. The Admin sends the invitation.
4. The system creates a single-use, expiring token.
5. The invitation email is sent.
6. The Company shows Invitation Pending.

### Alternatives and Errors

- Resend replaces or invalidates the earlier pending token.
- An active Client account blocks another invitation.
- Email-delivery failure is shown without falsely marking the invitation delivered.

### Postconditions

- At most one usable Client invitation exists for the Company.

---

## Flow 6.4 — Deactivate or Reactivate Client Access

**Actor:** Admin  
**Trigger:** The Admin changes Client access.  
**Preconditions:** A Client account exists.

### Main Flow

1. The Admin selects Deactivate or Reactivate.
2. Deactivation requires confirmation.
3. The system updates the account access state.
4. Active sessions are invalidated when access is deactivated.
5. The Company activity timeline updates.

### Alternatives and Errors

- An archived Company cannot reactivate Client access until restored.
- A failed update preserves the previous access state.

### Postconditions

- Workspace entry matches the account access state.

---

## Flow 6.5 — Update Pilot Status

**Actor:** Admin  
**Trigger:** The Admin edits Pilot status or milestones.  
**Preconditions:** The Company exists and is not archived.

### Main Flow

1. The Admin opens the Pilot section.
2. The Admin updates status, timeline, milestones, or participating talent.
3. The system validates and saves.
4. The Client-visible Pilot status updates.
5. A Client notification is created when appropriate.

### Alternatives and Errors

- Invalid milestone ordering is rejected.
- A failed update does not partially change the Client view.

### Postconditions

- Admin and Client views show the same current Pilot state.

---

## Flow 6.6 — Archive or Restore Company

**Actor:** Admin  
**Trigger:** The Admin selects Archive or Restore.  
**Preconditions:** The Company exists.

### Main Flow

1. Archiving requires confirmation and explains the access impact.
2. The Admin confirms.
3. The Company moves to Archived.
4. Client sessions and Workspace access are disabled.
5. From Archived Companies, the Admin may restore the Company.
6. The Company returns to an active administrative state.

### Alternatives and Errors

- Cancelling changes nothing.
- Restoring does not automatically reactivate a deactivated Client account.
- A failed lifecycle change preserves the previous state.

### Postconditions

- Company history is preserved.
- Client access follows the documented state.

---

# 7. Talent Recruitment and Profile Creation

## Flow 7.1 — Review Talent Application

**Actor:** Admin  
**Trigger:** The Admin opens a Talent Application.  
**Preconditions:** The application exists.

### Main Flow

1. The system displays candidate data, resume, current stage, assessment history, notes, and activity.
2. The Admin reviews the application.
3. The Admin adds internal notes or corrects permitted administrative information.
4. The system saves and records changes.

### Alternatives and Errors

- A private file load failure offers retry without exposing a public file URL.
- Failed saves preserve notes.

### Postconditions

- The application is ready for a stage decision.

---

## Flow 7.2 — Advance Recruitment Stage and Record Assessment

**Actor:** Admin  
**Trigger:** The Admin begins or completes a recruitment stage.  
**Preconditions:** The application is active and the transition is valid.

### Main Flow

1. The Admin selects the stage: Screening, Technical, English, or Remote Readiness.
2. The system shows the fields required for that stage.
3. The Admin records outcome, scores where applicable, and notes.
4. The Admin saves the assessment.
5. The system records the result and advances the application when passed.

### Alternatives and Errors

- The Admin may save incomplete work when the stage supports a draft.
- A failed or unsuitable outcome may branch to rejection.
- Invalid stage transitions are blocked.

### Postconditions

- Assessment history and current stage are accurate.

---

## Flow 7.3 — Reject Talent Application

**Actor:** Admin  
**Trigger:** The Admin selects Reject.  
**Preconditions:** The application is not already rejected or archived.

### Main Flow

1. The system requests a rejection reason and confirmation.
2. The Admin enters the reason and confirms.
3. The application becomes `REJECTED`.
4. Pending profile-completion tokens are invalidated.
5. Activity is recorded.

### Alternatives and Errors

- Cancelling leaves the application unchanged.
- A missing required reason blocks rejection.

### Postconditions

- The application is closed and cannot advance without a defined reopening action.

---

## Flow 7.4 — Approve Talent and Send Profile-Completion Request

**Actor:** Admin  
**Trigger:** The Admin approves a candidate.  
**Preconditions:** Required recruitment stages are complete.

### Main Flow

1. The system shows an approval summary.
2. The Admin confirms approval.
3. The application becomes `APPROVED`.
4. The Admin selects Request Profile Completion.
5. The system creates a single-use, expiring completion token.
6. The system sends the completion email.
7. The application becomes `PROFILE_COMPLETION_REQUESTED`.

### Alternatives and Errors

- Missing required assessments block approval.
- The Admin may approve now and send the request later.
- Resending replaces the earlier usable completion token.
- Email failure is shown and does not report delivery success.

### Postconditions

- The candidate is approved.
- No Talent Profile or account has been created.

---

## Flow 7.5 — Complete Tokenized Final-Information Form

**Actor:** Approved Talent acting through a secure token  
**Trigger:** The Talent opens the profile-completion link.  
**Preconditions:** The token is valid, unused, and linked to an approved application.

### Main Flow

1. The system validates the token before showing application context.
2. The Talent enters final professional information.
3. The Talent uploads or confirms a profile photo and resume.
4. The Talent submits the form.
5. The system validates fields and uploads.
6. The system stores the submission on the Talent Application.
7. The token is consumed.
8. The application becomes `PROFILE_INFORMATION_SUBMITTED`.
9. The Admin is notified.
10. The Talent sees confirmation that BlihOps will review the information.

### Alternatives and Errors

- Invalid, expired, replaced, or consumed tokens show an unavailable-link state.
- Validation errors retain valid form values.
- Upload or server failure does not consume the token.
- The form does not offer account Login because no Talent account exists yet.

### Postconditions

- Final information awaits Admin review.
- No Talent Profile or account is created automatically.

---

## Flow 7.6 — Review Submission and Manually Create Talent Profile

**Actor:** Admin  
**Trigger:** The Admin opens a Profile Information Submitted application.  
**Preconditions:** A valid completion submission exists and no Talent Profile is linked.

### Main Flow

1. The Admin compares the original application, assessments, and final-information submission.
2. The Admin corrects or completes profile information.
3. The Admin enters admin-only fields such as seniority, rates, verification, assessment summary, visibility, and internal notes.
4. The Admin previews the client-visible profile.
5. The Admin selects Create Talent Profile.
6. The system validates all required profile fields.
7. The system creates one Talent Profile linked to the application.
8. The application becomes `PROFILE_CREATED`.
9. The Talent Profile opens.

### Alternatives and Errors

- Missing information returns the Admin to the profile draft.
- A linked profile blocks duplicate creation.
- A failed transaction creates no partial profile.
- The Admin may save administrative preparation before final creation when supported.

### Postconditions

- One admin-created Talent Profile exists.
- No Talent account exists until separately invited.

---

## Flow 7.7 — Send or Resend Talent Account Invitation

**Actor:** Admin  
**Trigger:** The Admin selects Invite Talent from the created profile.  
**Preconditions:** The application is approved, the Talent Profile exists and is not archived, and no active Talent account exists.

### Main Flow

1. The Admin confirms the Talent email.
2. The system validates profile and account eligibility.
3. The Admin sends the invitation.
4. The system creates a single-use, expiring account token.
5. The invitation email is sent.
6. The profile shows Invitation Pending.

### Alternatives and Errors

- Resending replaces the earlier pending token.
- An active Talent account blocks a new account invitation.
- Missing or archived profile state blocks invitation.
- Email failure remains visible to Admin.

### Postconditions

- At most one usable Talent account invitation exists.

---

## Flow 7.8 — Archive or Restore Talent Application

**Actor:** Admin  
**Trigger:** The Admin selects Archive or Restore.  
**Preconditions:** The application exists.

### Main Flow

1. Archive requires confirmation.
2. The system moves the application to Archived and invalidates pending completion tokens.
3. From Archived Applications, the Admin may Restore.
4. The system returns the application to its prior meaningful stage when valid.

### Alternatives and Errors

- An application linked to an active Talent Profile remains linked; profile lifecycle is managed separately.
- A failed lifecycle update preserves the previous state.

### Postconditions

- The application appears in the correct view with history preserved.

---

# 8. Admin Talent Profile Management

## Flow 8.1 — Browse and View Talent Profiles

**Actor:** Admin  
**Trigger:** The Admin opens Talent Profiles.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system loads the selected Visible, Hidden, or Archived view.
2. The Admin searches, filters, sorts, or paginates.
3. The Admin selects a profile.
4. The system shows all professional, commercial, verification, visibility, internal, account, and activity information.

### Alternatives and Errors

- No results show an intentional empty state.
- A load failure offers retry.

### Postconditions

- The Admin can act on the selected profile.

---

## Flow 8.2 — Edit Talent Profile as Admin

**Actor:** Admin  
**Trigger:** The Admin selects Edit on a Talent Profile.  
**Preconditions:** The profile exists and is not archived, or archived editing is explicitly permitted.

### Main Flow

1. The system displays all editable profile fields.
2. The Admin updates professional or admin-only information.
3. The Admin saves.
4. The system validates the complete profile.
5. The system updates the profile and records Admin as actor.
6. Client-visible information updates immediately when the profile is visible.

### Alternatives and Errors

- Invalid fields are identified.
- Failed uploads or saves preserve the edit where safe.
- A concurrent update requires refresh or conflict resolution.

### Postconditions

- The Talent Profile reflects the Admin’s saved changes.

---

## Flow 8.3 — Publish or Hide Talent Profile

**Actor:** Admin  
**Trigger:** The Admin selects Publish or Hide.  
**Preconditions:** The profile exists and is not archived.

### Main Flow

1. For Publish, the system validates required client-visible fields.
2. The Admin confirms the visibility change.
3. The system updates visibility.
4. Visible profiles appear in the Client Talent Directory.
5. Hidden profiles are removed from Client Workspaces.
6. Activity is recorded.

### Alternatives and Errors

- Incomplete profiles cannot publish and list missing requirements.
- A failed update preserves the previous visibility.

### Postconditions

- Client visibility matches the profile state.

---

## Flow 8.4 — Archive or Restore Talent Profile

**Actor:** Admin  
**Trigger:** The Admin selects Archive or Restore.  
**Preconditions:** The profile exists.

### Main Flow

1. Archiving requires confirmation and explains the access impact.
2. The Admin confirms.
3. The system archives and hides the profile.
4. Talent Portal access and active Talent sessions are disabled.
5. From Archived Profiles, the Admin may restore the profile.
6. The profile returns as hidden.

### Alternatives and Errors

- Restoring does not automatically republish the profile.
- Restoring does not automatically reactivate a separately deactivated account.
- A failed lifecycle update preserves the previous state.

### Postconditions

- Visibility and Talent Portal access follow the documented lifecycle state.

---

# 9. Client Workspace

## Flow 9.1 — Enter Client Workspace

**Actor:** Client  
**Trigger:** The Client logs in or opens an authorized Workspace URL.  
**Preconditions:** The Client account, Company, and Workspace are active.

### Main Flow

1. The system validates the Client session and Company ownership.
2. The Workspace shell loads.
3. The Dashboard displays Company context, Pilot summary, available Talent summary, recent activity, and Interview Request updates.

### Alternatives and Errors

- Deactivated access or archived Company state blocks entry.
- A load failure shows a retryable Workspace error.

### Postconditions

- The Client is inside only their own Workspace.

---

## Flow 9.2 — Browse, Search, Filter, and Sort Talent

**Actor:** Client  
**Trigger:** The Client opens Talent Directory.  
**Preconditions:** The Client Workspace is active.

### Main Flow

1. The system loads active, visible Talent Profiles.
2. The Client enters search text or applies role, skills, experience, or availability filters.
3. The Client selects sorting and changes pages as needed.
4. Results update while preserving the chosen controls.
5. The Client selects a Talent Profile.

### Alternatives and Errors

- No matches show an empty state with Clear Filters.
- Hidden or archived profiles disappear from results.
- A request failure retains the current query and offers retry.

### Postconditions

- The Client reaches a relevant client-safe profile or keeps a reproducible result set.

---

## Flow 9.3 — View Talent Profile as Client

**Actor:** Client  
**Trigger:** The Client selects a Talent Profile.  
**Preconditions:** The profile is active and visible.

### Main Flow

1. The system validates Client Workspace access and profile visibility.
2. The profile displays client-safe professional, availability, verification, portfolio, resume, and rate information.
3. Internal notes, assessments, private contact data, and admin controls are omitted.
4. The Client may return to preserved Directory results or request an interview.

### Alternatives and Errors

- A profile hidden after navigation shows an unavailable state.
- Protected files require authorized access.

### Postconditions

- The Client has sufficient permitted information to decide on an Interview Request.

---

## Flow 9.4 — Request Interview

**Actor:** Client  
**Trigger:** The Client selects Request Interview on a Talent Profile.  
**Preconditions:** The Client account and Talent Profile are active.

### Main Flow

1. The request form identifies the Talent and Company.
2. The Client adds relevant context.
3. The Client reviews and submits.
4. The system validates eligibility.
5. The system creates an Interview Request with status `PENDING`.
6. Admin is notified.
7. The Client sees confirmation and a link to Interview Requests.

### Alternatives and Errors

- The system warns about an equivalent open request and prevents an accidental duplicate.
- A profile hidden before submission blocks creation and explains the change.
- A failed submission preserves context and offers retry.

### Postconditions

- One pending Interview Request exists for the Client’s Company and selected Talent Profile.

---

## Flow 9.5 — Track Interview Requests

**Actor:** Client  
**Trigger:** The Client opens Interview Requests.  
**Preconditions:** The Client Workspace is active.

### Main Flow

1. The system loads only the Company’s Interview Requests.
2. The Client filters or opens a request.
3. The detail shows Talent summary, status, submitted context, scheduling information, and client-visible updates.
4. The Client returns to the request list.

### Alternatives and Errors

- No requests show a link to Talent Directory.
- Cancelled and completed requests remain available as history.

### Postconditions

- The Client understands the current state of each request.

---

## Flow 9.6 — Track Pilot Progress

**Actor:** Client  
**Trigger:** The Client opens Pilot Status.  
**Preconditions:** The Client Workspace is active.

### Main Flow

1. The system loads the current Pilot state.
2. The Client reviews status, timeline, milestones, participating talent where applicable, and updates.

### Alternatives and Errors

- If no Pilot has started, the page explains the current pre-pilot state.
- A load failure offers retry.

### Postconditions

- The Client sees the latest Admin-managed Pilot information.

---

# 10. Talent Portal

## Flow 10.1 — View Own Profile

**Actor:** Talent  
**Trigger:** The Talent logs in or opens Talent Profile Management.  
**Preconditions:** The Talent account and linked profile are active.

### Main Flow

1. The system validates the Talent session and profile ownership.
2. The single Profile Management page loads.
3. The page groups professional information, skills and experience, links and resume, and availability.
4. Talent-editable fields are enabled.
5. Admin-only fields are omitted or displayed read-only only when useful.

### Alternatives and Errors

- Missing profile linkage blocks the page and directs the Talent to BlihOps.
- Archived profile state blocks editing.
- A load failure offers retry.

### Postconditions

- The Talent sees only their own permitted profile experience.

---

## Flow 10.2 — Edit Permitted Profile Information

**Actor:** Talent  
**Trigger:** The Talent edits Profile Management and selects Save.  
**Preconditions:** The Talent account and profile are active.

### Main Flow

1. The Talent edits permitted fields.
2. The system prevents changes to admin-only fields.
3. The Talent saves.
4. The system validates the submitted fields.
5. The system updates the Talent Profile immediately.
6. If the profile is client-visible, the permitted changes are immediately reflected for Clients.
7. The page shows success and the activity log identifies Talent as actor.

### Alternatives and Errors

- Invalid values show field-level feedback.
- A concurrent Admin update requires refresh before conflicting fields are overwritten.
- A failed save retains unsaved values and offers retry.

### Postconditions

- The permitted profile information reflects the Talent’s saved update.

---

## Flow 10.3 — Replace Profile Photo or Resume

**Actor:** Talent  
**Trigger:** The Talent selects an upload control.  
**Preconditions:** The Talent account and profile are active.

### Main Flow

1. The Talent chooses a file.
2. The system validates file type and size.
3. Upload progress is shown.
4. The upload completes and the new file is staged.
5. The Talent saves the profile.
6. The profile references the new file.
7. Storage cleanup follows the configured replacement policy.

### Alternatives and Errors

- Invalid files are rejected before upload.
- Upload failure preserves the previous file and other unsaved form values.
- Cancelling a staged replacement preserves the current profile file.

### Postconditions

- The profile uses the successfully saved photo or resume.

---

# 11. Managed Website Content

## Flow 11.1 — Manage Trusted Logos

**Actor:** Admin  
**Trigger:** The Admin opens Managed Content → Trusted Logos.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system loads logos in display order.
2. The Admin creates or edits name, logo image, optional website, and active state.
3. The Admin saves.
4. The Admin may reorder entries.
5. The system updates the public Home section with active entries in order.

### Alternatives and Errors

- Invalid image or URL fields block saving.
- Delete requires confirmation.
- Reorder failure restores the last saved order.

### Postconditions

- Home displays the saved active trusted logos in the correct order.

---

## Flow 11.2 — Manage Testimonials and Primary Testimonial

**Actor:** Admin  
**Trigger:** The Admin opens Managed Content → Testimonials.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system loads text and video testimonials in display order.
2. The Admin creates or edits the relevant type-specific fields.
3. The Admin may activate/deactivate and reorder testimonials.
4. The Admin selects one active testimonial as Primary.
5. The system validates that the Home content state has exactly one active Primary testimonial.
6. The system saves and updates the public Home sections.

### Alternatives and Errors

- Invalid media or missing type-specific fields block saving.
- Selecting a new Primary removes Primary from the previous item atomically.
- Deactivating or deleting the current Primary requires selecting a replacement.
- Delete requires confirmation.

### Postconditions

- Active testimonials appear in order.
- Exactly one active testimonial fills the primary managed-outsourcing placement.

---

## Flow 11.3 — Update Services Hero Media

**Actor:** Admin  
**Trigger:** The Admin opens Managed Content → Services Hero Media.  
**Preconditions:** The singleton content entry exists or can be initialized.

### Main Flow

1. The current video, cover image, accessible label, and update information load.
2. The Admin uploads or selects replacement media.
3. The system validates media types and sizes.
4. The Admin previews and saves.
5. The singleton object updates.
6. The Services hero uses the saved video and cover image.

### Alternatives and Errors

- Upload failure preserves current live media.
- Validation failure prevents publishing the replacement.
- Cancelling leaves the singleton unchanged.

### Postconditions

- Services uses one complete, valid hero media object.

---

## Flow 11.4 — Create or Edit Bilingual Case Study

**Actor:** Admin  
**Trigger:** The Admin creates or opens a Case Study.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The editor opens with English and German tabs plus shared metadata.
2. The Admin enters localized title, slug, summary, body, services, and outcomes.
3. The Admin enters shared client, category, image, tags, and featured state.
4. The Admin saves as Draft or selects Publish.
5. Draft validates only fields required to persist the draft.
6. Publish validates all required English, German, and shared fields.
7. The system checks locale-specific slug uniqueness.
8. The system saves one bilingual record.
9. Published content becomes available in both locales.

### Alternatives and Errors

- Incomplete locale content may remain Draft but cannot publish.
- A duplicate slug identifies the affected locale.
- Unpublish removes both locales together.
- Delete requires confirmation.
- Upload or save failures preserve editor content where safe.

### Postconditions

- The Case Study is either a saved Draft or published in both locales.

---

## Flow 11.5 — Create or Edit Bilingual Insight

**Actor:** Admin  
**Trigger:** The Admin creates or opens an Insight.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The editor opens with English and German tabs plus shared metadata.
2. The Admin enters localized title, slug, excerpt, and body.
3. The Admin enters shared author, category, tags, hero image, read time, and featured state.
4. The Admin saves as Draft or selects Publish.
5. Publish validates both locale tabs and shared fields.
6. The system checks locale-specific slug uniqueness.
7. The system saves one bilingual record.
8. Published content becomes available in both locales.

### Alternatives and Errors

- Incomplete translations can remain Draft only.
- Unpublish removes both locales together.
- Duplicate slug, upload, validation, and save errors identify the affected data.
- Delete requires confirmation.

### Postconditions

- The Insight is either a saved Draft or published in both locales.

---

## Flow 11.6 — Manage Careers Roles

**Actor:** Admin  
**Trigger:** The Admin opens Managed Content → Careers Roles.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system loads English-only roles.
2. The Admin creates or edits title, department, location, employment type, summary, overview, responsibilities, requirements, active state, and featured state.
3. The Admin saves.
4. Active roles appear publicly.
5. Inactive roles remain available to Admin but disappear publicly.

### Alternatives and Errors

- Required-field validation blocks saving.
- Delete requires confirmation.
- Deactivating a role removes its public detail.

### Postconditions

- Careers displays only active saved roles.

---

## Flow 11.7 — Manage Bilingual Pilot FAQs

**Actor:** Admin  
**Trigger:** The Admin opens Managed Content → Pilot FAQs.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system loads FAQs in display order.
2. The Admin creates or edits English and German question/answer pairs.
3. The Admin saves the entry.
4. The Admin may reorder or activate/deactivate entries.
5. Activation validates both locale pairs.
6. Active FAQs appear on the Pilot page in the requested locale and saved order.

### Alternatives and Errors

- An incomplete locale may be saved inactive but cannot activate.
- Delete requires confirmation.
- Reorder failure restores the last saved order.

### Postconditions

- Pilot displays only complete active FAQs in the correct locale and order.

---

# 12. Settings and Operational Oversight

## Flow 12.1 — Use Admin Dashboard

**Actor:** Admin  
**Trigger:** The Admin enters the Admin Portal.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The Dashboard loads operational counts and recent activity.
2. The Admin selects a widget or activity item.
3. The system opens the relevant filtered list or resource detail.

### Alternatives and Errors

- A failed widget shows a local retry without blocking other dashboard content.
- Empty metrics link to the relevant creation or intake context when useful.

### Postconditions

- The Admin reaches the operational work requiring attention.

---

## Flow 12.2 — Update Email Template

**Actor:** Admin  
**Trigger:** The Admin opens Settings → Email Templates.  
**Preconditions:** The template exists.

### Main Flow

1. The Admin selects a template.
2. The system shows subject, body, and allowed variables.
3. The Admin edits and previews the template.
4. The system validates variables.
5. The Admin saves.
6. Future emails use the updated template.

### Alternatives and Errors

- Unknown or missing required variables block saving.
- Failed save preserves the edit.

### Postconditions

- The selected template has one valid current version.

---

## Flow 12.3 — Review Calendly Configuration

**Actor:** Admin  
**Trigger:** The Admin opens Settings → Calendly.  
**Preconditions:** The Admin is authenticated.

### Main Flow

1. The system shows connection status, event mapping, webhook configuration state, and last webhook received.
2. The Admin reviews configuration health.
3. Permitted configuration changes are validated and saved.

### Alternatives and Errors

- Sensitive values remain masked.
- A failed connection check shows diagnostic status without exposing secrets.

### Postconditions

- The Admin understands the current Calendly integration state.

---

## Flow 12.4 — Update Interview Request as Admin

**Actor:** Admin  
**Trigger:** The Admin opens an Interview Request.  
**Preconditions:** The request exists.

### Main Flow

1. The system displays Company, Client-submitted context, Talent Profile, status, scheduling information, notes, and activity.
2. The Admin updates status or scheduling information.
3. The system validates and saves.
4. The Client-visible request updates.
5. A Client notification is created when appropriate.

### Alternatives and Errors

- Invalid status transitions are blocked.
- Failed saves preserve the previous request state.

### Postconditions

- Admin and Client views reflect the same current Interview Request status.

---

## 13. Cross-Flow Invariants

- Profile-completion submission never creates a Talent Profile or Talent account automatically.
- Talent account invitation is available only after Admin creates the Talent Profile.
- Talent enters Profile Management after authentication; there is no Talent Dashboard.
- Each Company has one Client account and one Client Workspace.
- Client Workspace contains no Team Management or additional-user invitation.
- Talent edits only permitted professional fields and those changes publish immediately.
- Admin remains the only role that can change rates, assessments, verification, visibility, lifecycle status, and internal notes.
- Bilingual Case Studies, Insights, and Pilot FAQs cannot publish or activate until English and German content is complete.
- Destructive actions require confirmation and preserve activity history where required.

