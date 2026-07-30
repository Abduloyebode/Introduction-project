> **AI-drafted technical reference.** This file covers the mechanics of the Step 1 auth setup (comparison detail, architecture, env vars, run/deploy steps). Reviewed and verified, but drafted with AI assistance rather than hand-written — see [`Authentication-Documentation.md`](./Authentication-Documentation.md) in this folder for the human write-up and library-choice reasoning.

## Decision summary

| Option | Verdict | Why |
|--------|---------|-----|
| Clerk | Not chosen | Users/sessions live outside our own Postgres; weak fit for a self-hosted Coolify setup and later `Workflow.owner` / org data living in our own DB |
| Auth.js | Runner-up | Well known, but email/password is a second-class path there (Credentials + JWT workarounds); the Auth.js team itself moved to Better Auth in late 2025 |
| **Better Auth** | **Chosen** | First-class email/password, sessions stored in Postgres, works cleanly on self-hosted Coolify |

### What was built (and what wasn't)

**Built**
- Email/password sign-up, sign-in, sign-out
- Protected `/dashboard` (cookie check in `proxy.ts` + a real server-side session check on the page itself)
- Prisma schema + migration for Better Auth's tables (`user`, `session`, `account`, `verification`)
- Zod validation with clear form errors
- Local Postgres via `docker-compose.yml`
- A Coolify-oriented Dockerfile that runs `prisma migrate deploy` on container start

**Not built (on purpose)**
- OAuth / magic links / password reset / 2FA
- Profile editing, roles, enforced email verification
- Anything that doesn't earn a place on the path to "account → dashboard" — matches the evaluator's minimalism feedback: no decorative controls.

## How the pieces connect

1. **Postgres** stores users, sessions, accounts, and verification rows.
2. **Prisma** is the ORM + migration tool (Prisma 7 uses a driver adapter: `@prisma/adapter-pg` + `pg`).
3. **Better Auth** handles password hashing, session cookies, and the `/api/auth/*` routes.
4. **Next.js App Router** serves `/sign-up`, `/sign-in`, `/dashboard`.

App code lives in the fork `Abduloyebode/demo-nextjs-app`, on `feature/step-1-auth` (merged into `main`).

## Environment variables

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | Postgres connection string |
| `BETTER_AUTH_SECRET` | Auth encryption/hashing secret (≥ 32 chars; e.g. `openssl rand -base64 32`) |
| `BETTER_AUTH_URL` | Public app URL (`http://localhost:3000` locally; the live HTTPS URL in Coolify) |
| `HOSTNAME` | Must be `0.0.0.0` in Coolify so the reverse proxy can reach the container |

See `.env.example` in the app repo. Never commit real secrets.

## Local run

```bash
# from demo-nextjs-app
docker compose up -d
cp .env.example .env   # then set BETTER_AUTH_SECRET
npm run db:deploy
npm run dev
```

Smoke test:
1. Open `/sign-up`, create an account → land on `/dashboard`
2. Sign out, sign in again via `/sign-in`
3. Private/incognito window on `/dashboard` → redirect to `/sign-in`

## Coolify deploy notes

1. Add a **PostgreSQL** resource in the same Coolify project (version 16, matching local).
2. Point the app at the branch that contains auth (`main`, once merged).
3. Use the **Dockerfile** build pack, not Nixpacks — Nixpacks ignores the custom `CMD` that runs the migration on startup. This tripped us up initially: the app deployed and looked fine, but the migration had silently never run because Coolify was auto-detecting a Nixpacks build instead of using the repo's Dockerfile.
4. Set `DATABASE_URL` (Coolify's **internal** DB host, not `localhost`), `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL` (the live HTTPS URL), and keep `HOSTNAME=0.0.0.0`.
5. Deploy and confirm live `/sign-up` → `/dashboard` actually writes to the database, not just that pages load.

One more real bug hit along the way: after switching to the Dockerfile build pack, the container crashed on startup with `Cannot find module 'effect'` — a transitive dependency of Prisma's config system that got missed because the Dockerfile's runner stage was copying only specific `node_modules` subfolders instead of the whole thing. Fixed by copying the full `node_modules` into the runner stage instead of cherry-picking packages by name.

Live URL used for this task: `https://uw752r62ktlsxq8p4y72rq97.89.167.4.27.sslip.io`

## Limitations / follow-ups

- Email verification is not required (no mail provider set up in Step 1).
- No password reset flow yet.
- Interactive weekly-rhythm card polish (removing the unused priority inputs) is parked until Step 1 is fully closed.
- Step 2+ will hang workflows off the same `user` table; Better Auth's organization plugin is a candidate for Step 5, not installed now.
