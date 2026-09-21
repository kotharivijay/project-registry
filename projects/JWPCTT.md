# JWPCTT Goods Received

| Field | Value |
|---|---|
| System | Desktop PC (DESKTOP-OJP04VH, Windows 10 Pro) |
| Version | 0.1.0 |
| Status | Live in production; actively adding features |
| Location | `C:\Users\vijay\JWPCTT` |
| Live site | https://jwpctt-kotharivijays-projects.vercel.app |
| Repo | https://github.com/kotharivijay/JWPCTT |
| Last updated | 2026-09-21 |

## Description

Web portal for **Jasol Water Pollution Control And Treatment Trust (JWPCTT)**
and its 74 connected member units to record goods received at each factory
(Date, Transport, LR No, Bales). Units sign up and wait for authoriser
approval before they can log in; each unit gets its own dashboard, and the
JWPCTT admin sees every unit and builds combined reports across all of them.

## Complete info

**Tech stack:** Next.js 14 (App Router, TypeScript strict), Tailwind CSS v3,
Prisma 5 ORM on a dedicated Neon Postgres database (project database name
`JWPCTT`, not the default `neondb`), NextAuth v4 with a Credentials provider
(login ID + password, no OAuth), bcryptjs for password hashing, `xlsx` for
Excel template generation/parsing, SWR on the client. Deployed on Vercel
(project `jwpctt`, org `kotharivijays-projects`).

**Auth model:** single `Unit` table doubles as both member units (`role:
UNIT`) and the trust's admin/authoriser (`role: ADMIN`). Signup creates a
`PENDING` unit; login is blocked until an admin approves it (`app/api/admin/
units/[id]/route.ts`). Admin login ID/password are seeded via `prisma/
seed.ts` from `ADMIN_LOGIN_ID`/`ADMIN_PASSWORD`/`ADMIN_NAME` env vars.

**Core flows:**
- `app/signup`, `app/login` — unit self-registration and gated sign-in.
- `app/admin/approvals` — approve/reject/deactivate units.
- `app/entry` — goods-received entry form (mobile + desktop), with
  localStorage + server-synced draft autosave (`app/api/draft`) and
  duplicate-LR warning (same Transport+LR within a unit).
- `app/entry`'s transport field is a searchable dropdown seeded with two
  default transports ("Raj Roadline", "Raj Transport Company") for every new
  unit (`lib/transports.ts`), plus an inline "+ Add as new transport" action
  backed by `POST /api/transports`.
- `app/upload` — Excel bulk upload: download template
  (`GET /api/upload/template`), review (`POST /api/upload/review`, parse-only,
  classifies rows as ready/dup_saved/dup_file/invalid), commit
  (`POST /api/upload/commit`, per-row Skip/Overwrite/Keep-both decisions,
  10-minute undo via `POST /api/upload/[id]/undo`).
- `app/dashboard` — a unit's own week/month/year stats.
- `app/admin/dashboard`, `app/admin/report` — all-units overview and a
  combined report (group by unit/transport/date) for the trust office.

**Known issues / next steps:**
- Commit route originally wrapped one Prisma query per row inside a single
  interactive transaction — failed with a generic "Server error" on files
  around 270+ rows (Prisma's default 5s transaction timeout). Fixed by
  bulk-inserting with `createMany` (see `app/api/upload/commit/route.ts`);
  verified against production with a 275-row file.
- **Not yet built:** an instant alert (requested: "missed call") to the
  admin whenever a new signup is waiting for approval. No telephony/SMS
  provider is wired up anywhere in this project — still needs a channel
  decision (Telegram bot is the free, low-effort option; a real phone call
  needs a paid provider like Exotel/Twilio, no free tier exists for that).
- **Not yet built:** an in-file Excel dropdown (data validation list) on the
  Transport Name column of the downloadable template.
- The seeded admin password is a placeholder (`change-me-please` at time of
  writing) — rotate it before wider rollout.
- Vercel **Deployment Protection (SSO)** was disabled project-wide so member
  units can reach the site without a Vercel login — intentional, but note it
  if the project ever needs to be locked back down.
- NextAuth crashes at build time if `NEXTAUTH_URL` is ever set to an empty
  string (not just unset) — this bit the very first deploy. Keep it either
  fully unset (Preview) or a real URL (Production), never `""`.

**Setup / run locally:**
```
cd C:\Users\vijay\JWPCTT
npm install
npm run db:push     # applies prisma/schema.prisma to DATABASE_URL
npm run db:seed     # creates the admin unit from ADMIN_* env vars
npm run dev
```
Required env vars (`.env.local`, gitignored): `DATABASE_URL`, `NEXTAUTH_URL`,
`NEXTAUTH_SECRET`, `ADMIN_LOGIN_ID`, `ADMIN_PASSWORD`, `ADMIN_NAME`,
`ADMIN_MOBILE`. Same six are set in Vercel's Production environment.

**Deploy:** push to `main` on GitHub auto-deploys via Vercel's Git
integration, or run `vercel --prod --yes` from the project folder (linked via
`vercel link --project jwpctt`).
