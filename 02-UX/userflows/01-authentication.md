# User Flow 1 — User Authentication

## Overview

This document defines the authentication flows for BlihOps. Authentication is invitation-based for Clients and Talents, while Administrators access the platform through the Admin Portal.

---

# User Flow 1.1 — Admin Login

## Overview

This flow describes how an Administrator signs in to the Admin Portal.

---

## Trigger

The administrator opens **admin.blihops.com** and selects **Log In**.

---

## Preconditions

- The administrator account exists.
- The administrator is not authenticated.

---

## Main Flow

1. The administrator opens the login page.
2. The administrator enters:
   - Email address
   - Password
3. The administrator submits the form.
4. The system validates the credentials.
5. The administrator is authenticated.
6. An authenticated session is created.
7. The administrator is redirected to the Admin Dashboard.

---

## Alternative Flows

### Existing Session

1. The administrator already has a valid session.
2. The system skips the login page.
3. The Admin Dashboard is displayed.

---

## Error Flows

### Invalid Credentials

1. The administrator enters an incorrect email or password.
2. The system displays an authentication error.
3. The administrator remains on the login page.

---

### Server Error

1. Authentication cannot be completed.
2. The system displays an error message.
3. The administrator can retry.

---

## Postconditions

- The administrator is authenticated.
- An active session exists.
- The Admin Dashboard is displayed.

---

# User Flow 1.2 — Client Invitation Acceptance

## Overview

This flow describes how a company representative accepts an invitation to access their Client Workspace.

---

## Trigger

The client opens an invitation link received by email.

---

## Preconditions

- A valid invitation exists.
- The invitation has not expired.
- The associated company exists.

---

## Main Flow

1. The client opens the invitation link.
2. The system validates the invitation.
3. If the client is not authenticated, the system requests password creation.
4. The client creates a password.
5. The system creates the client's account.
6. The invitation is marked as accepted.
7. The client is authenticated.
8. The client is redirected to:

   `/workspace/{workspaceId}`

---

## Alternative Flows

### Existing Client Account

1. The invited email already has an account.
2. The client signs in.
3. The invitation is accepted automatically.
4. The Client Workspace opens.

---

## Error Flows

### Invalid Invitation

1. The invitation is invalid or expired.
2. The system displays an appropriate message.
3. The client cannot continue.

---

### Server Error

1. The invitation cannot be processed.
2. The system displays an error.
3. The client may retry later.

---

## Postconditions

- The client account is active.
- The invitation is consumed.
- The Client Workspace is accessible.

---

# User Flow 1.3 — Talent Invitation Acceptance

## Overview

This flow describes how an approved Talent activates their Talent Portal after receiving an invitation.

---

## Trigger

The Talent opens the invitation email.

---

## Preconditions

- The Talent has been approved.
- A valid invitation exists.

---

## Main Flow

1. The Talent opens the invitation link.
2. The system validates the invitation.
3. If the Talent is not authenticated, password creation is required.
4. The Talent creates a password.
5. The system activates the Talent account.
6. The invitation is marked as accepted.
7. The Talent is authenticated.
8. The Talent is redirected to:

   `/talent/{talentId}`

---

## Alternative Flows

### Existing Talent Account

1. The Talent already has an account.
2. The Talent signs in.
3. The invitation is accepted.
4. The Talent Portal opens.

---

## Error Flows

### Invitation Expired

1. The invitation has expired.
2. The system informs the Talent.
3. Access is denied until a new invitation is issued.

---

### Server Error

1. Activation cannot be completed.
2. The system displays an error.
3. The Talent may retry later.

---

## Postconditions

- The Talent account is active.
- The invitation is consumed.
- The Talent Portal is accessible.

---

# User Flow 1.4 — User Login

## Overview

This flow describes how existing Clients and Talents sign in after their accounts have been activated.

---

## Trigger

The user selects **Log In**.

---

## Preconditions

- The account has already been activated.
- The user is not authenticated.

---

## Main Flow

1. The user opens the login page.
2. The user enters:
   - Email address
   - Password
3. The user submits the form.
4. The system validates the credentials.
5. The user is authenticated.
6. The system determines the user's account type.
7. The user is redirected to:

   - `/workspace/{workspaceId}` for Clients
   - `/talent/{talentId}` for Talents

---

## Alternative Flows

### Existing Session

1. A valid session already exists.
2. The system skips authentication.
3. The appropriate dashboard opens.

---

## Error Flows

### Invalid Credentials

1. The user enters incorrect credentials.
2. The system displays an authentication error.

---

### Server Error

1. Authentication cannot be completed.
2. The system displays an error.
3. The user can retry.

---

## Postconditions

- The user is authenticated.
- An active session exists.
- The appropriate portal is displayed.

---

# User Flow 1.5 — Reset Password

## Overview

This flow describes how authenticated users recover access when they forget their password.

---

## Trigger

The user selects **Forgot Password**.

---

## Preconditions

- The account exists.

---

## Main Flow

1. The user enters their email address.
2. The system verifies the account.
3. A password reset email is sent.
4. The user opens the reset link.
5. The user enters a new password.
6. The system validates the password.
7. The password is updated.
8. The user signs in with the new password.

---

## Error Flows

### Invalid Reset Link

1. The reset link has expired or is invalid.
2. The system requests a new password reset.

---

### Server Error

1. Password reset cannot be completed.
2. The system displays an error.
3. The user may retry.

---

## Postconditions

- The password is updated.
- Previous reset links become invalid.
- The user can authenticate with the new password.

---

# User Flow 1.6 — Logout

## Overview

This flow describes how authenticated users end their session.

---

## Trigger

The user selects **Logout**.

---

## Preconditions

- The user is authenticated.

---

## Main Flow

1. The user opens the account menu.
2. The user selects **Logout**.
3. The system invalidates the session.
4. The user is redirected to the appropriate login page.

---

## Error Flows

### Server Error

1. Session termination cannot be confirmed.
2. The client clears the local session.
3. The user is redirected to the login page.

---

## Postconditions

- The session is terminated.
- Protected pages are no longer accessible.
- Authentication is required for future access.