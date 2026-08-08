# Deployment

Deployment architecture and runbook for the blihops platform.

## 1. Architecture

```
+----------------+        +------------------+        +-------------------+
|  blihops-web   |  HTTPS |   api.blihops.com | HTTPS  |    Render (PaaS)  |
|  (Vercel)      | -----> |   Render service  | -----> |   blihops-api     |
|  frontend      |        |   (Express/TS)    |        |   + PostgreSQL    |
+----------------+        +------------------+        |   on Neon (BaaS)  |
                                                       +-------------------+
```

| Layer      | Platform         | Why                                                        |
| ---------- | ---------------- | ---------------------------------------------------------- |
| Frontend   | Vercel           | Already live for `blihops-web`; zero-ops static/SSR deploy |
| Backend    | Render (free)    | Long-running Node process, git auto-deploy, TLS, health checks |
| Database   | Neon (free)      | Managed Postgres 17, branching, auto-backups, pooled connections |

The company normally deploys to VPS. This project deliberately uses managed
services so the team ships features instead of running Postgres and TLS.
A VPS migration path is preserved (see §11) — the same Docker Compose used
locally runs on a VPS unchanged.

## 2. Region

Team and users are in the Horn of Africa. Reality check on free tiers:

- **Render free tier** runs instances in the **US (Oregon)** only. Expect
  ~200-250 ms extra latency from the Horn; acceptable for early stage.
  Paid tier adds Frankfurt (eu-central-1), which is the closest major hub.
- **Neon free tier** supports **Frankfurt (eu-central-1)** — pick this region
  for the database. Closest major infra, keeps DB latency reasonable.

When the company budget allows, move the backend to a paid Render instance in
Frankfurt (same code, zero changes).

## 3. Backend setup (Render)

Create a **Web Service** from the `blihops-api` GitHub repo (`main` branch):

| Setting            | Value                                                       |
| ------------------ | ----------------------------------------------------------- |
| Environment        | Node                                         |
| Build command      | `pnpm install --frozen-lockfile && pnpm build`              |
| Start command      | `pnpm start` (runs `node dist/server.js`)    |
| Release command    | `prisma migrate deploy` (runs before deploy completes)      |
| Health check path  | `/health`                                                   |
| Instance type      | Free                                                       |
| Auto-deploy        | On pushes to `main`                                        |

Notes:

- Node **`>=24.18 <25`** is required (engines in `package.json`) — confirm the
  Render Node runtime supports 24 before creating the service.
- `pnpm build` already runs `prisma generate` (via `pnpm db:generate`), so no
  extra generate step is needed in the build command.
- `prisma migrate deploy` in the release command applies committed migrations
  safely (never `prisma db push` in production).
- The server already logs pino JSON to stdout — Render captures it under
  **Logs** in the dashboard.
- Transactional email is sent through **Resend** from the verified domain
  `mail.blihops.com` (`EMAIL_FROM`). Resend is a separate service — its env
  vars are listed in §5.

### Custom domain

1. Settings → Custom Domain → add `api.blihops.com`.
2. Point DNS at Render: add a `CNAME` record `api` → `<service>.onrender.com`
   (or an `A` record + `ALIAS` per Render's instructions).
3. Render provisions the TLS certificate automatically; the API sends HSTS via
   helmet already.

## 4. Database setup (Neon)

1. Create a Neon project, region **eu-central-1 (Frankfurt)**, database
   `blihops`.
2. Copy both connection strings:
   - **Pooled** (runtime): `postgresql://...@ep-xxx.pooler.eu-central-1.aws.neon.tech/blihops?pgbouncer=true&sslmode=require`
   - **Direct** (migrations/admin): `postgresql://...@ep-xxx.eu-central-1.aws.neon.tech/blihops?sslmode=require`
3. Set them in Render as `DATABASE_URL` (pooled) and `DIRECT_URL` (direct).

Migrations and Prisma CLI commands use `DIRECT_URL` (`prisma.config.ts`
resolves `DIRECT_URL ?? DATABASE_URL`); the runtime app always uses
`DATABASE_URL` (pooled). If `DIRECT_URL` is unset, the CLI falls back to
`DATABASE_URL` (local dev setup).

### Branching

Neon branches give every PR a disposable database:

- `main` branch ↔ production database.
- Create a branch per PR/preview environment; `prisma migrate deploy` against
  it in the preview service.

Backups are automatic (7-day point-in-time on free tier).

## 5. Environment variables

Set these in the Render service dashboard (and in local `.env` for dev). The
schema in `src/shared/configs/envSchema.ts` is the source of truth.

| Variable             | Local dev (Docker)                          | Production (Render)                        |
| -------------------- | ------------------------------------------- | ------------------------------------------ |
| `NODE_ENV`           | `development`                               | `production`                               |
| `PORT`               | `4000`                                      | `10000` (Render-injected)                  |
| `API_URL`            | `http://localhost:4000`                     | `https://api.blihops.com`                  |
| `WEB_URL`            | `http://localhost:3000`                     | `https://blihops.com`                      |
| `DATABASE_URL`       | `postgresql://blihops:blihops@localhost:5432/blihops` | Neon **pooled** URL        |
| `DIRECT_URL`         | (same as local)                             | Neon **direct** URL                        |
| `LOG_LEVEL`          | `debug`                                     | `info`                                     |
| `CORS_ORIGIN`        | `http://localhost:3000,http://localhost:5173` | `https://blihops.com,https://www.blihops.com,https://admin.blihops.com` |
| `BETTER_AUTH_SECRET` | random ≥32 chars                            | random ≥32 chars                           |
| `RESEND_API_KEY`     | Resend API key                              | Resend API key                             |
| `EMAIL_FROM`         | `Blih Ops <noreply@mail.blihops.com>`       | same                                       |
| `EMAIL_LOGO_URL`     | `https://blihops.com/logo.png`              | same (hosted on the deployed web app)      |
| `SEED_ADMIN_PASSWORD`| required for `pnpm seed:admin`              | **not set in production**                  |
| `SEED_DEMO_PASSWORD` | required for `pnpm seed:demo`               | **not set in production**                  |

Notes:

- `CORS_ORIGIN` is comma-separated (parsed by the env schema) and also feeds
  Better Auth `trustedOrigins` (already wired — no separate config). It must
  contain every frontend origin: the public site (`blihops.com`), its `www`
  alias, and the admin console (`admin.blihops.com`).
- `DATABASE_URL` (pooled) is used at runtime by the app; `DIRECT_URL`
  (direct) is used by Prisma CLI/migrations via `prisma.config.ts`.
- `RESEND_API_KEY` is required outside the test environment (enforced by the
  env schema). Emails are sent from the verified Resend domain
  `mail.blihops.com`.
- `BETTER_AUTH_SECRET` replaced the earlier `JWT_SECRET` placeholder. Sign-up
  is disabled (`disableSignUp`), so production user creation happens only via
  admin invites and the admin seed — which is why seed passwords are never
  set in production.
- Never commit `.env` files; CI already fails on committed env files.

## 6. Frontend (Vercel)

Set both API env vars to `https://api.blihops.com` in the `blihops-web` (and
`blihops-admin`) project settings:

- `API_URL` — server-side only, used by the `proxy.ts` middleware to validate
  sessions against the API.
- `NEXT_PUBLIC_API_URL` — inlined into the client bundle, used by the
  better-auth client and `apiFetch`.

- CORS on the API already allows the configured origins; add the Vercel
  preview domain (`*.vercel.app`) to `CORS_ORIGIN` only if previews must call
  the API cross-origin.

## 7. CI/CD

- GitHub Actions (`ci.yml`) stays quality-only: lint, typecheck, format,
  build, tests. It never touches production.
- Deploys are platform-driven: push to `main` → Render auto-deploys
  (build → migrate → start). Vercel auto-deploys the web app.
- Optional later: Neon branch + Render preview service for PR environments.

## 8. Free-tier caveats (known, accepted)

| Caveat                        | Impact                                            | Mitigation                                  |
| ----------------------------- | ------------------------------------------------- | ------------------------------------------- |
| Render free sleeps after ~15 min idle | First request after idle = cold start (several seconds) | `/health` hits by uptime pings; upgrade when it matters |
| Neon free scales compute to zero | First DB query after idle is slow (~1-2 s)      | Same as above; upgrade path exists          |
| Render free = US region only  | Higher latency from the Horn                     | Move to paid Frankfurt later                |
| Free tier monthly limits      | 750 instance-hours (Render), 190 compute-hours (Neon) | Plenty for a pilot; monitor usage    |

## 9. Costs

- Vercel: $0 (free tier) — already in use.
- Render: $0 (free tier).
- Neon: $0 (free tier).
- Resend: $0 (free tier, 100 emails/day) — plenty for a pilot.
- **Total ≈ $0/month** until the pilot needs paid tiers.

## 10. Backups & DR

- Neon: automatic 7-day point-in-time recovery. Restore drill: create a branch
  from a point in time and verify the API boots against it (`DATABASE_URL`
  swapped).
- Render: deploys are git-reversible (`main` history), no data stored on the
  service (stateless; all state in Neon).
- Logs: pino JSON captured in Render logs; export to Axiom/Betterstack later
  if needed.

## 11. VPS fallback path

If the company mandates self-hosting:

1. Run the same `docker-compose.yml` used locally (`postgres:17-alpine` + app
   container) on the VPS.
2. Terminate TLS with Caddy (auto certs) in front of the app.
3. Move `DATABASE_URL`/`DIRECT_URL` to the VPS-hosted Postgres; the schema and
   migrations are identical, so `prisma migrate deploy` applies unchanged.

No code changes are required — this is why local dev uses Docker Compose.

## 12. Rollout checklist

- [ ] Neon project created (Frankfurt), pooled + direct URLs copied
- [ ] Render service created from repo, build/start/release commands set
- [ ] Env vars from §5 set in Render (incl. `BETTER_AUTH_SECRET`, Resend vars)
- [ ] No `SEED_*` vars set in production
- [ ] `api.blihops.com` CNAME + TLS verified (`curl https://api.blihops.com/health`)
- [ ] Vercel web + admin project envs set to `https://api.blihops.com`
- [ ] CORS smoke test from the live web domain
- [ ] First `prisma migrate deploy` runs clean on production (via `DIRECT_URL`)
- [ ] Resend domain `mail.blihops.com` verified + `EMAIL_LOGO_URL` reachable
- [ ] Prod email smoke test: real invite/reset email sent, logo renders
