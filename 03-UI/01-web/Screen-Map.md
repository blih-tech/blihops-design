# Web UI Screen Map

Design source: `web.pen`

## Canvas Sections

| Section | Content | Initial status |
|---|---|---|
| W00 | Foundations and reusable components | Complete |
| W01 | Authentication and access | Complete |
| W02 | Invitations and profile completion | Planned |
| W03 | Client Workspace shell and dashboard | Planned |
| W04 | Talent discovery and interview request | Planned |
| W05 | Talent Pod planning | Planned |
| W06 | Interview Requests and Pilot | Planned |
| W07 | Talent Profile Management | Planned |
| W08 | Responsive and shared system states | Planned |

## W01 — Authentication and Access

| Screen ID | URL | Variant |
|---|---|---|
| AUTH-02 | `/en/sign-in` | Default |
| AUTH-02 | `/en/sign-in` | Validation errors |
| AUTH-02 | `/en/sign-in` | Authentication failed |
| AUTH-03 | `/en/forgot-password` | Default |
| AUTH-04 | `/en/forgot-password` | Request submitted |
| AUTH-05 | `/en/reset-password?token={token}` | Default |
| AUTH-05 | `/en/reset-password?token={token}` | Validation errors |
| AUTH-06 | `/en/reset-password?token={token}` | Invalid or expired token |
| AUTH-13 | `/en/workspaces/{workspaceId}` | Access denied |
| AUTH-14 | `/en/workspaces/{workspaceId}` | Workspace not found |

Expired sessions redirect directly to `/en/sign-in`; they do not use a dedicated screen.

## W02 — Invitations and Profile Completion

| Screen ID | URL | Variant |
|---|---|---|
| AUTH-07 | `/en/accept-invitation/client?token={token}` | Default |
| AUTH-08 | `/en/accept-invitation/talent?token={token}` | Default |
| AUTH-09 | Invitation URL | Invalid or expired |
| AUTH-10 | Invitation URL | Account already activated |
| COMPFORM-01 | `/en/complete-profile?token={token}` | Default |
| COMPFORM-01 | `/en/complete-profile?token={token}` | Validation errors |
| COMPFORM-02 | Completion URL | Photo upload in progress |
| COMPFORM-03 | Completion URL | Resume upload in progress |
| COMPFORM-04 | Completion URL | Invalid or expired token |
| COMPFORM-05 | Completion URL | Submission success |

The completion form submits final information for Admin review. It does not
create a Talent account.

## W03 — Client Workspace

| Screen ID | URL | Variant |
|---|---|---|
| CLIENT-01 | `/en/workspace` | Workspace shell |
| CLIENT-02 | `/en/workspace` | Dashboard |
| CLIENT-03 | `/en/workspace` | Notifications open |
| CLIENT-19 | `/en/workspace` | Workspace unavailable |

## W04 — Talent Discovery

| Screen ID | URL | Variant |
|---|---|---|
| CLIENT-04 | `/en/workspace/talent` | Default |
| CLIENT-04 | `/en/workspace/talent` | Loading |
| CLIENT-05 | `/en/workspace/talent?search={query}&sort={order}` | Search and sort |
| CLIENT-06 | `/en/workspace/talent` | Filters open |
| CLIENT-07 | `/en/workspace/talent` | No matching talent |
| CLIENT-08 | `/en/workspace/talent/{talentId}` | Default |
| CLIENT-09 | Talent Profile URL | Resume viewer open |
| CLIENT-10 | Talent Profile URL | Profile unavailable |
| CLIENT-26 | `/en/workspace/talent` | Add to Pod open |
| CLIENT-26 | `/en/workspace/talent/{talentId}` | Add to Pod open |
| CLIENT-11 | Talent Profile URL | Request interview open |
| CLIENT-12 | Talent Profile URL | Duplicate request warning |
| CLIENT-13 | Talent Profile URL | Request submitted |

## W05 — Talent Pod Planning

| Screen ID | URL | Variant |
|---|---|---|
| CLIENT-20 | `/en/workspace/pods` | Default |
| CLIENT-21 | `/en/workspace/pods` | Empty |
| CLIENT-22 | `/en/workspace/pods` | Create Pod open |
| CLIENT-23 | `/en/workspace/pods/{podId}` | Default |
| CLIENT-24 | `/en/workspace/pods/{podId}` | Edit Pod open |
| CLIENT-25 | `/en/workspace/pods/{podId}` | Add members open |
| CLIENT-27 | `/en/workspace/pods/{podId}` | Select or clear Pod Lead |
| CLIENT-28 | `/en/workspace/pods/{podId}` | Remove member confirmation |
| CLIENT-29 | `/en/workspace/pods/{podId}` | Archive confirmation |
| CLIENT-30 | `/en/workspace/pods/{podId}` | Unavailable member |

## W06 — Interview Requests and Pilot

| Screen ID | URL | Variant |
|---|---|---|
| CLIENT-14 | `/en/workspace/interview-requests` | Default |
| CLIENT-16 | `/en/workspace/interview-requests` | Empty |
| CLIENT-15 | `/en/workspace/interview-requests/{requestId}` | Default |
| CLIENT-17 | `/en/workspace/pilot` | Active Pilot |
| CLIENT-18 | `/en/workspace/pilot` | No active Pilot |

## W07 — Talent Profile Management

| Screen ID | URL | Variant |
|---|---|---|
| TALENT-01 | `/en/profile` | Talent Portal shell |
| TALENT-02 | `/en/profile` | Default |
| TALENT-07 | `/en/profile` | Photo upload open |
| TALENT-08 | `/en/profile` | Resume upload open |
| TALENT-09 | `/en/profile` | Saved |
| TALENT-10 | `/en/profile` | Validation errors |
| TALENT-11 | `/en/profile` | Concurrent update |
| TALENT-12 | `/en/profile` | Access blocked |

## W08 — Shared Design States

Shared variants are added to the most representative primary screen rather
than repeated for every route:

- Route and list loading
- Mutation and upload progress
- Save success
- Retryable server error
- Empty filtered results
- Destructive confirmation
- Stale-data conflict
- Keyboard focus
- Mobile navigation
- Narrow form layout
