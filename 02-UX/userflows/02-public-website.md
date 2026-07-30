# User Flow 2 — Lead Management

## Overview

This document defines how administrators manage company leads throughout the sales pipeline, from initial inquiry to company creation or archival.

---

# User Flow 2.1 — Create Lead

## Overview

This flow describes how a new lead enters the BlihOps platform.

---

## Trigger

A lead is submitted through the website or manually created by an administrator.

---

## Preconditions

- The lead does not already exist.

---

## Main Flow

### Website Submission

1. A company representative submits the pilot request form.
2. The system validates the submitted information.
3. A new lead is created.
4. The lead is assigned the **New** status.
5. The lead appears in the Admin Portal.

---

### Manual Creation

1. An administrator opens the **Leads** page.
2. The administrator selects **Create Lead**.
3. The administrator enters the lead information.
4. The system validates the data.
5. The lead is created.
6. The lead appears in the Leads list.

---

## Alternative Flows

### Existing Company

1. The submitted company already exists.
2. The system warns the administrator.
3. The administrator decides whether to continue or cancel.

---

## Error Flows

### Invalid Information

1. Required fields are missing or invalid.
2. Validation errors are displayed.
3. The administrator or visitor corrects the information.

---

### Server Error

1. The lead cannot be created.
2. The system displays an error.
3. The operation can be retried.

---

## Postconditions

- A new lead exists.
- The lead appears in the sales pipeline.

---

# User Flow 2.2 — Review Lead

## Overview

This flow describes how administrators review and update lead information.

---

## Trigger

The administrator opens a lead.

---

## Preconditions

- The lead exists.

---

## Main Flow

1. The administrator opens the Leads page.
2. The administrator selects a lead.
3. The system displays:
   - Company information
   - Contact information
   - Notes
   - Current status
   - Activity history
4. The administrator updates the information if necessary.
5. The changes are saved immediately.

---

## Alternative Flows

### Add Internal Notes

1. The administrator records meeting notes.
2. The notes become part of the lead history.

---

## Error Flows

### Server Error

1. The lead cannot be loaded or updated.
2. The system displays an error.
3. The administrator can retry.

---

## Postconditions

- Lead information reflects the latest updates.

---

# User Flow 2.3 — Update Lead Status

## Overview

This flow describes how administrators move a lead through the sales pipeline.

---

## Trigger

The administrator changes the lead status.

---

## Preconditions

- The lead exists.

---

## Main Flow

1. The administrator opens a lead.
2. The administrator selects a new status.
3. The system validates the transition.
4. The lead status is updated.
5. The pipeline reflects the change immediately.

---

## Example Statuses

- New
- Contacted
- Discovery Scheduled
- Qualified
- Converted
- Closed Lost

---

## Alternative Flows

### Reopen Lead

1. A previously closed lead becomes active again.
2. The administrator changes the status.
3. The lead returns to the active pipeline.

---

## Error Flows

### Invalid Status Transition

1. The selected transition is not permitted.
2. The system rejects the update.

---

### Server Error

1. The update cannot be completed.
2. The administrator can retry.

---

## Postconditions

- The lead reflects its latest pipeline status.

---

# User Flow 2.4 — Convert Lead to Company

## Overview

This flow describes how a qualified lead becomes an active client company.

---

## Trigger

The administrator selects **Convert to Company**.

---

## Preconditions

- The lead has been qualified.
- The company does not already exist.

---

## Main Flow

1. The administrator opens a qualified lead.
2. The administrator selects **Convert to Company**.
3. The system creates a Company.
4. A Client Workspace is created.
5. The lead is marked as **Converted**.
6. The administrator can immediately invite client users.

---

## Alternative Flows

### Continue Later

1. The administrator creates the Company.
2. Client invitations are sent later.

---

## Error Flows

### Company Already Exists

1. A company with the same identity already exists.
2. The conversion is blocked.

---

### Server Error

1. Conversion cannot be completed.
2. The administrator may retry.

---

## Postconditions

- A Company exists.
- A Client Workspace exists.
- The lead is marked as converted.

---

# User Flow 2.5 — Archive Lead

## Overview

This flow describes how administrators archive leads that are no longer active.

---

## Trigger

The administrator selects **Archive**.

---

## Preconditions

- The lead exists.

---

## Main Flow

1. The administrator opens a lead.
2. The administrator selects **Archive**.
3. The system requests confirmation.
4. The administrator confirms.
5. The lead is moved to the Archived Leads list.

---

## Alternative Flows

### Restore Lead

1. The administrator opens Archived Leads.
2. The administrator restores the lead.
3. The lead returns to the active pipeline.

---

## Error Flows

### Server Error

1. The archive operation fails.
2. The administrator can retry.

---

## Postconditions

- The lead is no longer visible in the active pipeline.
- The lead remains available for future reference.