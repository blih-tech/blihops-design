# User Flow 5 — Talent Recruitment

## Overview

This document defines how administrators manage the talent recruitment pipeline from application submission to approval or rejection.

---

# User Flow 5.1 — Submit Talent Application

- Trigger: A candidate submits the application form.
- Main Flow:
  1. Complete the application.
  2. Upload required documents.
  3. Submit.
  4. System validates.
  5. Application enters the recruitment pipeline.
- Postconditions:
  - Application is created.

---

# User Flow 5.2 — Review Application

- Trigger: Administrator opens an application.
- Main Flow:
  1. Review candidate information.
  2. Review uploaded resume.
  3. Record notes.
  4. Continue or reject.

---

# User Flow 5.3 — Screening

- Trigger: Administrator starts screening.
- Main Flow:
  1. Review qualifications.
  2. Record screening outcome.
  3. Move to the next stage.

---

# User Flow 5.4 — Technical Assessment

- Trigger: Candidate reaches technical evaluation.
- Main Flow:
  1. Assign assessment.
  2. Review results.
  3. Record evaluation.
  4. Continue or reject.

---

# User Flow 5.5 — English Assessment

- Trigger: Candidate passes technical review.
- Main Flow:
  1. Schedule assessment.
  2. Record results.
  3. Continue or reject.

---

# User Flow 5.6 — Remote Readiness Assessment

- Trigger: Candidate passes previous stages.
- Main Flow:
  1. Review equipment.
  2. Review internet connection.
  3. Evaluate communication.
  4. Record outcome.

---

# User Flow 5.7 — Approve Talent

- Trigger: Candidate passes all stages.
- Main Flow:
  1. Approve candidate.
  2. System creates a Talent Profile.
  3. Candidate becomes eligible for client visibility.

---

# User Flow 5.8 — Reject Talent

- Trigger: Candidate fails any stage.
- Main Flow:
  1. Reject candidate.
  2. Record rejection reason.
  3. Close application.

---

# User Flow 5.9 — Send Talent Invitation

- Trigger: Candidate is approved.
- Main Flow:
  1. Send invitation.
  2. Email is delivered.
  3. Candidate activates their Talent Portal.

---

## Postconditions

- The candidate is either approved with a Talent Profile or rejected with a recorded outcome.