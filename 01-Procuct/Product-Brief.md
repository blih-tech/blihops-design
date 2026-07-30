# BlihOps V1 – Product Brief

# 1. Overview

BlihOps is an outsourcing platform that connects global companies with pre-vetted Ethiopian software engineers.

The first version focuses on validating the business by generating leads, managing operations internally, and providing approved clients with access to a curated talent pool.

This version intentionally excludes the future Skills and Talent Marketplace products, which will be developed in Version 2.

---

# 2. Goals

## Business Goals

- Generate qualified company leads.
- Build and maintain a vetted talent pool.
- Reduce the time required to match clients with engineers.
- Deliver a professional client experience during the pilot process.

## Product Goals

- Manage all leads from one admin portal.
- Recruit and validate engineers before clients need them.
- Provide approved companies with a private workspace to explore available talent.
- Keep internal operations simple and scalable.

---

# 3. Products

## 3.1 BlihOps Web

### Purpose

Public-facing website for marketing, lead generation, talent applications, and client access.

### Features

- Marketing pages
- Services
- About
- Contact
- Request Pilot
- Book a Call (Calendly)
- Join Talent Pool
- Client Workspace (Protected)

---

## 3.2 BlihOps Admin

### Purpose

Internal platform used by the BlihOps team to manage operations.

### Features

- Dashboard
- Lead Management
- Pilot Requests
- Contact Requests
- Booked Calls
- Talent Applications
- Talent Pool
- Company Management
- Rate Cards
- CMS
- User Management

---

## 3.3 BlihOps API

### Purpose

Backend powering both the public website and the admin platform.

### Responsibilities include:

- Authentication
- Lead Management
- Talent Management
- Company Management
- Client Workspace
- CMS
- Email & Invitations

---

# 4. User Types

## Visitor

Public website visitor.

### Can:

- Browse website
- Contact BlihOps
- Request a pilot
- Book a call
- Apply to the talent pool

---

## Client

Approved company invited by BlihOps.

### Can:

- Access Client Workspace
- Browse available talent
- View rate cards
- Save candidates
- Request interviews
- Track pilot progress

---

## Admin

Internal BlihOps team.

Can manage every aspect of the platform.

---

# 5. Talent Recruitment Flow

```text
Join Talent Pool
      ↓
Submit Application
      ↓
Application Review
      ↓
Screening Call
      ↓
Technical Assessment
      ↓
English Assessment
      ↓
Remote Readiness Assessment
      ↓
Internal Approval
      ↓
Create Rate Card
      ↓
Available in Talent Pool
```

---

# 6. Client Flow

```text
Visit Website
      ↓
Contact / Pilot Request / Book Call
      ↓
Lead Created
      ↓
Discovery Call
      ↓
Qualified
      ↓
Pilot Approved
      ↓
Create Client Workspace
      ↓
Invite Sent
      ↓
Client Activates Account
      ↓
Access Client Workspace
```

---

# 7. Client Workspace (V1)

## Purpose

Provide approved companies with a secure, private portal to explore available engineers.

### Features

- Dashboard
- Available Talent
- Talent Profiles
- Rate Cards
- Saved Candidates
- Interview Requests
- Pilot Status

---

# 8. Talent Pool

The Talent Pool is an internal database of approved engineers.

Each profile includes:

- Profile Photo
- Name
- Role
- Seniority
- Tech Stack
- Years of Experience
- English Level
- Availability
- Portfolio
- Resume
- Hourly Rate
- Monthly Rate
- Status

Only approved talents are visible to clients.

---

# 9. Architecture

```text
blihops-web
      │
┌───────────────┴───────────────┐
│                               │
Public Website          Client Workspace
            │
            ▼
      blihops-api
            ▲
            │
      blihops-admin
```

---

# 10. Out of Scope (Version 1)

The following products are intentionally postponed:

- BlihOps Skills
- BlihOps Talent Marketplace
- Public talent marketplace
- Self-service company registration
- Self-managed talent profiles
- AI talent matching
- Payments & invoicing
- Messaging system

These will be considered for Version 2 after validating the core outsourcing workflow.