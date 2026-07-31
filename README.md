# BlihOps Design

The official planning and design repository for BlihOps.

This repository is the product and UX source of truth used before UI design, technical planning, and implementation. Application source code is maintained separately.

---

## Repository Structure

```text
blihops-design/
├── 01-Procuct/
│   ├── Product-Brief.md
│   ├── PRD.md
│   └── historical PDF references
│
├── 02-UX/
│   ├── User-Personas.md
│   ├── Information-Archtecture.md
│   ├── Navigations.md
│   └── User-Flows.md
│
├── 03-ui/
│   ├── wireframes/
│   ├── mockups/
│   ├── exports/
│   ├── Design System.md
│   └── Components.md
│
├── 04-engineering/
│   ├── Implementation Plan.md
│   ├── Architecture.md
│   ├── Database Schema.md
│   ├── API Specification.md
│   └── Technical Decisions.md
│
└── assets/
```

Some later-stage folders and files are created only when that design stage begins.

---

## Canonical Document Order

When documents differ, resolve them in this order:

1. `01-Procuct/Product-Brief.md`
2. `01-Procuct/PRD.md`
3. `02-UX/User-Personas.md`
4. `02-UX/Information-Archtecture.md`
5. `02-UX/Navigations.md`
6. `02-UX/User-Flows.md`
7. Screen Inventory and later UI artifacts

The Markdown Product Brief and PRD are canonical. PDFs in `01-Procuct` are historical references and may not reflect the current V1 scope.

---

## Current V1 Product Boundaries

- One Client account and one Client Workspace per Company.
- No Client Team Management.
- Talent Profiles are created manually by Admin after recruitment and final-information review.
- Talent receives a separate account invitation after the profile exists.
- Talent Portal contains Profile Management only; it has no dashboard.
- Talent may edit permitted professional fields but not commercial, assessment, verification, visibility, lifecycle, or internal fields.
- Website content management is limited to the structured content types defined in the PRD.

---

## Design Workflow

1. Product discovery
2. Product Brief
3. Product Requirements Document
4. Personas and information architecture
5. Navigation
6. Consolidated User Flows
7. Screen Inventory
8. UI design
9. Technical planning
10. Engineering handoff

Each stage should be reviewed before the next stage begins.

---

## Related Application Repository

The existing BlihOps application monorepo is maintained outside this design directory. It may be inspected to verify current routes, content types, and behavior, but design-document updates do not modify application code unless implementation work is separately authorized.

---

## Philosophy

Plan first. Build second.

Every implementation should be backed by clear product requirements, deliberate UX structure, consistent UI, and documented technical decisions.

