# Step 1 — Auth library choice and setup notes

I spent a bit of time looking at how this app is actually hosted before picking anything. We're on Coolify with our own Postgres, not on a managed Vercel Marketplace setup, and the next steps (workflows owned by users, then orgs) need real user rows in that database. So I wanted auth that lives next to the app data, not in a separate SaaS user store.

I compared Clerk, Auth.js (NextAuth), and Better Auth against that. Clerk would have been fast for UI, but users and sessions would sit outside our Postgres and make later ownership awkward. Auth.js is well known, but email/password is a second-class path there (Credentials + JWT workarounds). Better Auth treated email/password and DB sessions as the normal case, which matched Step 1 as written.

So I went with Better Auth + PostgreSQL + Prisma. Below is what I actually did, what I deliberately skipped, and how to run it.

---

## Decision summary

| Option | Verdict | Why |
|--------|---------|-----|
| Clerk | No | Users live in Clerk; weak fit for Coolify + later `Workflow.owner` / org data in our DB |
| Auth.js | Runner-up | Familiar, but credentials/email-password is awkward; project is now under Better Auth |
| **Better Auth** | **Chosen** | First-class email/password, sessions in Postgres, works on self-hosted Coolify |

### What we built (and what we did not)

**Built**
- Email/password sign-up, sign-in, sign-out
- Protected `/dashboard` (optimistic cookie check in `proxy.ts` + real session check on the page)
- Prisma schema + migration for Better Auth tables (`user`, `session`, `account`, `verification`)
- Zod validation and clear form errors
- Local Postgres via `docker-compose.yml`
- Coolify-oriented Dockerfile that runs `prisma migrate deploy` on start

**Not built (on purpose)**
- OAuth / magic links / password reset / 2FA
- Profile editing, roles, email verification requirement
- Anything that does not earn a place on the path to “account → dashboard”

That matches the evaluator push for ruthless minimalism: no decorative controls.

---

## How the pieces connect

1. **Postgres** stores users, sessions, accounts, verification rows.
2. **Prisma** is the ORM + migration tool (Prisma 7 uses a driver adapter: `@prisma/adapter-pg` + `pg`).
3. **Better Auth** handles hashing (scrypt), session cookies, and the `/api/auth/*` routes.
4. **Next.js App Router** serves `/sign-up`, `/sign-in`, `/dashboard`.

App work lives on the fork: `Abduloyebode/demo-nextjs-app`, branch history on `feature/step-1-auth` / `main`.

---

## Environment variables

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | Postgres connection string |
| `BETTER_AUTH_SECRET` | Auth encryption/hashing secret (≥ 32 chars; use `openssl rand -base64 32`) |
| `BETTER_AUTH_URL` | Public app URL (`http://localhost:3000` locally; live HTTPS URL in Coolify) |
| `HOSTNAME` | Must be `0.0.0.0` in Coolify so Traefik can reach the container |

See `.env.example` in the app repo. Never commit real secrets.

---

## Local run

```bash
# from demo-nextjs-app
docker compose up -d
cp .env.example .env   # then set BETTER_AUTH_SECRET
npx prisma migrate deploy
npm run dev
```

Smoke test:
1. Open `/sign-up`, create an account → land on `/dashboard`
2. Sign out, sign in again via `/sign-in`
3. Private window on `/dashboard` → redirect to `/sign-in`

---

## Coolify deploy notes

1. Add a **PostgreSQL** resource in the same Coolify project.
2. Point the app at the branch that contains auth (fork `main` or `feature/step-1-auth`).
3. Prefer **Dockerfile** build pack (migrate runs on container start).
4. Set `DATABASE_URL` (Coolify **internal** DB host, not `localhost`), `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL` (live HTTPS URL), and keep `HOSTNAME=0.0.0.0`.
5. Deploy and confirm live `/sign-up` → `/dashboard`.

Live URL used for this task: `https://uw752r62ktlsxq8p4y72rq97.89.167.4.27.sslip.io`

---

## Limitations / follow-ups

- Email verification is not required (no mail provider in Step 1).
- No password reset yet.
- Interactive weekly-rhythm polish (removing unused priority inputs) is still parked until Step 1 is fully closed.
- Step 2+ will hang workflows off the same `user` table; org plugins in Better Auth are a later option for Step 5, not installed now.
