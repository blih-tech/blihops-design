# User Flow 7 — Client Workspace

## Overview

This document defines how client users interact with the Client Workspace after accepting their invitation.

---

# User Flow 7.1 — Enter Workspace

## Overview

This flow describes how a client enters their workspace.

---

## Trigger

The client signs in.

---

## Preconditions

- The client account is active.
- The client belongs to a company.

---

## Main Flow

1. The client signs in.
2. The system authenticates the client.
3. The Client Workspace loads.
4. The Dashboard is displayed.

---

## Postconditions

- The client is inside their workspace.

---

# User Flow 7.2 — Browse Talent Directory

## Overview

This flow describes how clients browse available talent.

---

## Trigger

The client opens the Talent Directory.

---

## Preconditions

- At least one Talent Profile is published.

---

## Main Flow

1. Open the Talent Directory.
2. Browse available talent.
3. View summary information.
4. Select a profile.

---

## Postconditions

- Talent profiles are available for review.

---

# User Flow 7.3 — Search & Filter Talent

## Overview

This flow describes how clients locate suitable talent.

---

## Trigger

The client uses Search or Filters.

---

## Main Flow

1. Enter a search query.
2. Apply filters such as:
   - Skills
   - Experience
   - Availability
3. Results update immediately.
4. Open a profile.

---

## Alternative Flows

### No Results

1. No matching talent is found.
2. An empty state is displayed.

---

## Postconditions

- Matching talent is displayed.

---

# User Flow 7.4 — View Talent Profile

## Overview

This flow describes how clients review a talent profile.

---

## Trigger

The client selects a profile.

---

## Main Flow

1. Open the profile.
2. View:
   - Bio
   - Skills
   - Experience
   - Resume
   - Availability
3. Decide whether to request an interview.

---

## Postconditions

- The client has enough information to continue.

---

# User Flow 7.5 — Request Interview

## Overview

This flow describes how clients request interviews.

---

## Trigger

The client selects **Request Interview**.

---

## Main Flow

1. Open a Talent Profile.
2. Select **Request Interview**.
3. Confirm the request.
4. The system creates an Interview Request.
5. Administrators are notified.

---

## Postconditions

- A new Interview Request exists.

---

# User Flow 7.6 — Manage Interview Requests

## Overview

This flow describes how clients monitor interview requests.

---

## Main Flow

1. Open Interview Requests.
2. View request status.
3. Review updates.
4. View completed interviews.

---

## Postconditions

- Interview requests reflect their latest status.

---

# User Flow 7.7 — Track Pilot Progress

## Overview

This flow describes how clients monitor their pilot engagement.

---

## Main Flow

1. Open Pilot Status.
2. Review progress.
3. Review assigned talent.
4. Monitor milestones.

---

## Postconditions

- Pilot progress is visible.

---

