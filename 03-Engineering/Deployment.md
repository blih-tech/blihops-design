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
| Build command      | `pnpm install --frozen-lockfile && pnpm build && prisma generate` |
| Start command      | `pnpm start` (runs `node dist/server.js`)    |
| Release command    | `prisma migrate deploy` (runs before deploy completes)      |
| Health check path  | `/health`                                                   |
| Instance type      | Free                                                       |
| Auto-deploy        | On pushes to `main`                                        |

Notes:

- `prisma generate` must run in the build because Prisma Client is generated
  code and the install command does not generate it.
- `prisma migrate deploy` in the release command applies committed migrations
  safely (never `prisma db push` in production).
- The server already logs pino JSON to stdout — Render captures it under
  **Logs** in the dashboard.

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

### Branching

Neon branches give every PR a disposable database:

- `main` branch ↔ production database.
- Create a branch per PR/preview environment; `prisma migrate deploy` against
  it in the preview service.

Backups are automatic (7-day point-in-time on free tier).

## 5. Environment variables

Set these in the Render service dashboard (and in local `.env` for dev).

| Variable       | Local dev (Docker)                          | Production (Render)                        |
| -------------- | ------------------------------------------- | ------------------------------------------ |
| `NODE_ENV`     | `development`                               | `production`                               |
| `PORT`         | `4000`                                      | `10000` (Render-injected)                  |
| `API_URL`      | `http://localhost:4000`                     | `https://api.blihops.com`                  |
| `DATABASE_URL` | `postgresql://blihops:blihops@localhost:5432/blihops` | Neon **pooled** URL        |
| `DIRECT_URL`   | (same as local)                             | Neon **direct** URL                        |
| `LOG_LEVEL`    | `debug`                                     | `info`                                     |
| `CORS_ORIGIN`  | `http://localhost:3000,http://localhost:5173` | `https://blihops.com,https://www.blihops.com` |
| `JWT_SECRET`   | random ≥32 chars                            | random ≥32 chars (replaced by Better Auth secrets when auth lands) |

Notes:

- `CORS_ORIGIN` is comma-separated (parsed by the env schema).
- The same origins list must go into Better Auth `trustedOrigins` once auth is
  wired.
- Never commit `.env` files; CI already fails on committed env files.

## 6. Frontend (Vercel)

- Set `API_URL=https://api.blihops.com` (and any `NEXT_PUBLIC_API_URL`) in the
  `blihops-web` project settings.
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
- [ ] Env vars from §5 set in Render
- [ ] `api.blihops.com` CNAME + TLS verified (`curl https://api.blihops.com/health`)
- [ ] Vercel web project env set to `https://api.blihops.com`
- [ ] CORS smoke test from the live web domain
- [ ] First `prisma migrate deploy` runs clean on production
