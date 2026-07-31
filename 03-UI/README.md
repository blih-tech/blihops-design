# BlihOps UI Design

This directory contains high-fidelity UI design work derived from:

- `../01-Procuct/Product-Brief.md`
- `../01-Procuct/PRD.md`
- `../02-UX/User-Flows.md`
- `../02-UX/Screen-Inventory.md`
- `../../blihops-web/DESIGN.md`

The deprecated `blihops-platform` repository is not a design or implementation
source.

## Directory Structure

```text
03-UI/
├── README.md
├── 01-web/
│   ├── Screen-Map.md
│   └── web-ui.pen
├── 02-admin/
│   ├── README.md
│   └── admin-ui.pen
└── 03-exports/
    └── README.md
```

`web-ui.pen` is the single source for the public-web application surfaces:
shared foundations, authentication and onboarding, Client Workspace, and the
Talent Portal. Keeping these screens together allows components and visual
rules to remain consistent.

`admin-ui.pen` is a separate source because the Admin Portal is a larger,
operations-focused product with its own shell and denser interaction patterns.
Admin design begins only after the web UI is reviewed.

## Canvas Organization

Each Pencil file uses a bounded grid rather than a long horizontal strip.

```text
Row 1  Foundations and shared components
Row 2  Primary route screens
Row 3  Secondary route screens
Row 4  Overlays, form states, and system variants
Row 5  Responsive reference screens
```

Related screens are collected inside labelled section boards. A board should
normally contain no more than four desktop frames per row.

## Screen Annotation

Every screen frame starts with a small annotation block:

```text
URL: /en/workspace/talent
Variant: Default
Screen ID: CLIENT-04
```

Rules:

- Use `/en` as the representative locale in designs. The equivalent `/de`
  route is implied.
- Use route parameters such as `{talentId}` and token placeholders rather than
  realistic IDs or secrets.
- Loading, empty, error, modal, and validation states retain the underlying
  URL and receive a `Variant` label.
- Add a query parameter only when it represents real navigable application
  state, such as directory search or filters.
- Do not invent separate routes solely to display a design variant.

## Design Source Precedence

When sources differ, follow this order:

1. Product behaviour in the PRD and User Flows.
2. Surface coverage in the Screen Inventory.
3. Visual direction in `blihops-web/DESIGN.md`.
4. Implemented tokens and shared primitives in `blihops-web`.

## Review Order

1. Shared foundations and application shells
2. Authentication and password recovery
3. Invitations and Talent final-information submission
4. Client Workspace
5. Talent Portal
6. Responsive and system-state pass
7. Admin Portal

