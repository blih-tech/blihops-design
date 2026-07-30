# User Flow 4 — Company Management

## Overview

This document defines how administrators manage client companies after a lead has been converted. It covers company information, client access, and company lifecycle management.

---

# User Flow 4.1 — Create Company

## Overview

This flow describes how an administrator manually creates a company.

---

## Trigger

The administrator selects **Create Company**.

---

## Preconditions

- The company does not already exist.

---

## Main Flow

1. The administrator opens the Companies page.
2. The administrator selects **Create Company**.
3. The administrator enters the company information.
4. The system validates the data.
5. The company is created.
6. A Client Workspace is created.
7. The company appears in the Companies list.

---

## Error Flows

### Duplicate Company

1. The company already exists.
2. The system prevents creation.

### Server Error

1. The company cannot be created.
2. The administrator can retry.

---

## Postconditions

- A new Company exists.
- A Client Workspace exists.

---

# User Flow 4.2 — View Company

## Overview

This flow describes how administrators review company information.

---

## Trigger

The administrator opens a company.

---

## Preconditions

- The company exists.

---

## Main Flow

1. Open the Companies page.
2. Select a company.
3. The system displays:
   - Company information
   - Client users
   - Pilot status
   - Interview requests
   - Activity history

---

## Postconditions

- Company information is available.

---

# User Flow 4.3 — Update Company

## Overview

This flow describes how administrators update company information.

---

## Trigger

The administrator edits a company.

---

## Preconditions

- The company exists.

---

## Main Flow

1. Open a company.
2. Select **Edit**.
3. Update one or more fields.
4. Save changes.
5. The system validates and updates the company.

---

## Error Flows

### Validation Error

1. Invalid information is entered.
2. The system displays validation errors.

### Server Error

1. The update fails.
2. The administrator can retry.

---

## Postconditions

- Company information reflects the latest changes.

---

# User Flow 4.4 — Invite Client User

## Overview

This flow describes how administrators invite company representatives.

---

## Trigger

The administrator selects **Invite Client**.

---

## Preconditions

- The company exists.

---

## Main Flow

1. Open the company.
2. Select **Invite Client**.
3. Enter the user's email.
4. Submit the invitation.
5. The system creates an invitation.
6. An invitation email is sent.

---

## Alternative Flows

### Resend Invitation

1. A pending invitation exists.
2. The administrator resends it.

---

## Error Flows

### Existing User

1. The user already belongs to the company.
2. The system prevents duplicate invitations.

---

## Postconditions

- A client invitation exists.

---

# User Flow 4.5 — Manage Client Users

## Overview

This flow describes how administrators manage client users.

---

## Trigger

The administrator opens the Client Users section.

---

## Preconditions

- At least one client user exists.

---

## Main Flow

1. View client users.
2. Update user information if needed.
3. Deactivate or reactivate access.
4. Save changes.

---

## Error Flows

### Server Error

1. The requested action fails.
2. The administrator can retry.

---

## Postconditions

- Client access reflects the latest changes.

---

# User Flow 4.6 — Archive Company

## Overview

This flow describes how administrators archive inactive companies.

---

## Trigger

The administrator selects **Archive**.

---

## Preconditions

- The company exists.

---

## Main Flow

1. Open the company.
2. Select **Archive**.
3. Confirm the action.
4. The company is moved to Archived Companies.

---

## Alternative Flows

### Restore Company

1. Open Archived Companies.
2. Restore the company.
3. The company becomes active again.

---

## Postconditions

- The company is archived while preserving its history.