# User Flow 3 — Lead Management

## Overview

This document defines how administrators manage company leads throughout the sales pipeline, from initial inquiry to company creation or archival.

---

# User Flow 3.1 — Create Lead

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
3. The user corrects the information.

---

### Server Error

1. The lead cannot be created.
2. The system displays an error message.
3. The operation can be retried.

---

## Postconditions

- A new lead exists.
- The lead appears in the sales pipeline.

---

# User Flow 3.2 — Review Lead

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

1. The administrator opens the **Leads** page.
2. The administrator selects a lead.
3. The system displays:
   - Company information
   - Contact information
   - Lead source
   - Notes
   - Current status
   - Activity history
4. The administrator updates the information.
5. The changes are saved immediately.

---

## Alternative Flows

### Add Internal Notes

1. The administrator records meeting notes.
2. The notes are added to the lead history.

---

## Error Flows

### Server Error

1. The lead cannot be loaded or updated.
2. The system displays an error message.
3. The administrator can retry.

---

## Postconditions

- The lead reflects the latest information.

---

# User Flow 3.3 — Update Lead Status

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
5. The updated status is reflected in the pipeline immediately.

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

1. The administrator reopens a previously closed lead.
2. The lead returns to the active pipeline.

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

- The lead reflects its current pipeline status.

---

# User Flow 3.4 — Convert Lead to Company

## Overview

This flow describes how a qualified lead becomes an active client company.

---

## Trigger

The administrator selects **Convert to Company**.

---

## Preconditions

- The lead is qualified.
- The company does not already exist.

---

## Main Flow

1. The administrator opens a qualified lead.
2. The administrator selects **Convert to Company**.
3. The system creates a Company.
4. The system creates a Client Workspace.
5. The lead status is updated to **Converted**.
6. The administrator is redirected to the Company details page.
7. The administrator can invite client users.

---

## Alternative Flows

### Invite Later

1. The company is created successfully.
2. Client invitations are sent at a later time.

---

## Error Flows

### Company Already Exists

1. A matching company already exists.
2. The conversion is blocked.

---

### Server Error

1. Conversion cannot be completed.
2. The administrator can retry.

---

## Postconditions

- A Company exists.
- A Client Workspace exists.
- The lead is marked as converted.

---

# User Flow 3.5 — Archive Lead

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
3. The system displays a confirmation dialog.
4. The administrator confirms the action.
5. The lead is moved to the Archived Leads list.

---

## Alternative Flows

### Restore Lead

1. The administrator opens the Archived Leads list.
2. The administrator restores the lead.
3. The lead returns to the active pipeline.

---

## Error Flows

### Server Error

1. The archive operation cannot be completed.
2. The administrator can retry.

---

## Postconditions

- The lead is removed from the active pipeline.
- The lead remains available in Archived Leads.