# Authentication and Access UI Revision

## Scope

Revise the existing web UI foundations and W01 authentication screens without changing the 1360 × 900 desktop frame size.

## Compact layout

- Keep all desktop shell and screen frames at 1360 × 900.
- Reduce internal padding, vertical gaps, header heights, and unused whitespace in the Auth, Client Workspace, and Talent Portal shells.
- Reduce the authentication card to approximately 1120 × 620.
- Use a 35/65 authentication split:
  - Brand panel: approximately 392 px.
  - Form panel: approximately 728 px.
- Keep form controls readable and preserve space for validation and recovery messages.

## Session expiry

- Remove the dedicated Session Expired screen and dialog.
- When a protected request detects an expired session, redirect directly to Sign In.
- Sign In may show a compact informational message explaining that the session expired.
- After successful authentication, return the user to the originally requested route when it remains valid and authorized.

## Access denied

Trigger: an authenticated user requests an existing workspace they are not authorized to access.

- Do not render the destination workspace shell, workspace name, workspace ID, or other destination data.
- User-facing title: `You don't have access to this workspace.`
- Supporting copy should explain that the signed-in account cannot open the workspace without mentioning roles, ownership checks, or implementation details.
- Show one action only: `Go back`.
- `Go back` returns to the previous safe BlihOps page. If no safe application history exists, it uses the signed-in user's valid entry route as an invisible fallback.

## Workspace not found

Trigger: the requested workspace ID does not resolve to a workspace.

- Do not render a workspace shell, workspace name, workspace ID, or diagnostic information.
- User-facing title: `We couldn't find this workspace.`
- Supporting copy should state that the link may be incorrect or the workspace may no longer be available.
- Show one action only: `Go back`.
- `Go back` returns to the previous safe BlihOps page. If no safe application history exists, it uses the signed-in user's valid entry route as an invisible fallback.

## Documentation alignment

- Remove the Session Expired screen from the web Screen Map and represent it as redirect behavior.
- Update the Screen Inventory so session expiry is a redirect, not a dialog.
- Rename the W01 resource-unavailable screen to Workspace Not Found and define it specifically around an unresolved workspace ID.
- Preserve Access Denied as the response for an existing but unauthorized workspace.

## Acceptance criteria

- Auth shell uses a visually clear 35/65 split.
- All existing shells and W01 screens have tighter internal spacing without clipping.
- No dedicated Session Expired frame remains.
- Access Denied and Workspace Not Found use neutral BlihOps presentation rather than the destination workspace shell.
- Both access states contain only one visible action: `Go back`.
- No implementation, authorization-model, workspace-ownership, or private-resource information appears in user-facing copy.
