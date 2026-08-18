# Web app development — KB ingest pack (2026-08-18)

Self-contained lessons from **places-agent MVP-1** admin UI delivery. Category: **web-app-development**. No secrets. Product AC stays in `1.places-agent/agent-specs/`; this pack is reusable engineering knowledge.

kb writes are **propose → human confirm**. Source of truth in repo: `knowledge/web-app-development/lessons-from-places-agent-mvp1.md`.

---

## Ingest manifest

Propose as **one category** with optional split later if kb search needs finer tags.

| Proposed kb title | Type | Ingest? | Notes |
| --- | --- | --- | --- |
| **Web app development — auth forms and token-in-email flows** | ops-lesson | ✅ Yes | One POST path; `bindFormSubmit`; URL leak redirect; `/login/fresh` route handler; CSRF + LAN origins |
| **Web app development — Next.js 16 admin app constraints** | ops-lesson | ✅ Yes | Custom server `main()`; Edge vs Node crypto split; TS7/lint gap; Turbopack/process hygiene; Prisma env |
| **Web app development — cross-machine local dev** | ops-lesson | ✅ Yes | `PUBLIC_BASE_URL`, `ALLOWED_DEV_ORIGINS`, `0.0.0.0`; operator confirmation gate |
| **Web app development — Playwright operator E2E** | ops-lesson | ✅ Yes | Python harness; testid waits; session clear; popups; `make db`; port conflicts; journey matrix |
| **Web app development — admin mail and destructive mutations** | ops-lesson | ✅ Yes | Resend; dev outbox fallback; delete guards; informational notification mail |
| **Web app development — i18n and dual auth testing** | ops-lesson | ✅ Yes | Four locales; key-based tests; session vs caller key separation |
| **Web app development — DoD gates for admin slices** | ops-lesson | ✅ Optional | Checklist + anti-patterns; overlaps workspace DoD — ingest if kb lacks a web-app DoD note |

---

## Consolidated content (ingest body)

### Auth forms and token-in-email flows

- Use **one** submission path: RHF + `bindFormSubmit` + POST JSON to `/api/admin/*`. Do not maintain server action and API route in parallel.
- Native GET submit puts credentials in the URL and drops invite tokens → false expired state. Fix with `preventDefault`, not layered sessionStorage patches.
- Server redirect strips leaked query keys (`password`, profile fields) as a safety net.
- E2E must assert URL has no `password=` after invite submit.
- Existing session cookie makes `/login` redirect to landing — clear via **`GET /login/fresh`** Route Handler (`cookies().delete()` fails in Server Components).
- Mutating admin APIs: CSRF Origin check; allow LAN IP origins in dev via env.

### Next.js 16 admin app constraints

- Custom `server.ts`: wrap `app.prepare()` in async `main()` when `tsx` compiles as CJS.
- Edge middleware: no `node:crypto`; keep cookie names edge-safe; verify sessions in Node handlers.
- `eslint-config-next` may not support TypeScript 7 — use `tsc --noEmit` until toolchain catches up.
- Dev stability: one dev process; `make down`; clear `.next` if Turbopack cache corrupts.
- Prisma CLI does not read `.env.local` — pass `DATABASE_URL` explicitly on migrate/seed.

### Cross-machine local dev

- Mail links need `PUBLIC_BASE_URL=http://<lan-ip>:<port>` so a second device can open invite/reset URLs.
- `ALLOWED_DEV_ORIGINS=<lan-ip>` (comma-separated) so Next dev serves JS from LAN Origin.
- Server must listen on `0.0.0.0` for LAN access.
- Do not mark invite/auth done without operator confirmation on this path.

### Playwright operator E2E

- Dynamic app: start real server (`make db` first).
- Prefer `data-testid` over English copy (i18n product).
- Avoid relying on `networkidle` with `next dev`; wait on test ids after `domcontentloaded`.
- `/login/fresh` before login tests; `expect_popup()` for `_blank` links.
- Minimum journeys: invite accept → login; home → instructions; login fail; issue key → HTTP tool call; locale switch.
- Tool behavior: Vitest HTTP/MCP, not admin UI.

### Admin mail and destructive mutations

- Prod: Resend failure blocks mutation (e.g. delete admin).
- Dev: in-memory outbox when no key; non-prod fallback to outbox when Resend fails.
- Mail copy from i18n keys; absolute URLs from `PUBLIC_BASE_URL`.
- Destructive UI: confirm dialog, disabled submit during request, in-dialog errors, server-side guards (no self-delete, no last admin).

### i18n and dual auth

- Catalogs: EN, CN, HK, TW — separate HK/TW; fallback locale → EN → key.
- Tests assert keys/test ids, not one language.
- Admin session ≠ caller API key; test boundaries separately.

### DoD gates (admin web slice)

- Vitest + tsc + Playwright critical paths + user acceptance.
- Do not ship token forms without E2E URL-leak proof.
- Known MVP-1 gaps: lint (TS7), coverage unmeasured, delete-admin E2E optional.

---

## Already in kb (do not duplicate)

| kb title | Relationship |
| --- | --- |
| places-agent Next.js 16 local runtime (if ingested from 2026-08-17 pack) | **Supersede or merge** with “Next.js 16 admin app constraints” above |
| Agent-mate admin visual | Keep separate — visual design, not process |
| ADR-012, ADR-018 | Decisions — link only, do not re-ingest as lessons |

## Do not ingest

- Story build order `14 → 15 → …` (lives in agent-stories; will rot)
- `.env.local` values or example secrets
- Mockup HTML files
- Per-feature AC text (use agent-specs)

## Tags for kb

`web-app-development`, `next`, `playwright`, `auth`, `i18n`, `places-agent`, `operator-ui`
