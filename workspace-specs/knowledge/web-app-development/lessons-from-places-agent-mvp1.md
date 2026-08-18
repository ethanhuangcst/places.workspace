---
title: Web app development lessons — places-agent MVP-1
type: ops-lesson
status: active
as_of: 2026-08-18
tags:
  - web-app-development
  - next
  - playwright
  - auth
  - i18n
  - places-agent
related_spec: 1.places-agent/agent-specs/4.test-strategy.md
related:
  - ../kb-ingest/web-app-development-2026-08-18.md
  - ../../adr/ADR-012-admin-ui-on-agent.md
  - ../../adr/ADR-018-mvp-by-capability.md
  - 1.places-agent/agent-specs/7.auth-refactor-plan.md
---

# Web app development lessons — places-agent MVP-1

Consolidated engineering lessons from delivering MVP-1 admin UI (Features 14–19) on Next.js 16 App Router. Reuse on other `*.agent-mate.ai` operator apps.

---

## 1. Auth forms and token-in-email flows

### One submission path

Token-in-email flows (invite accept, password reset/set) must use **one** browser → server path. Maintaining both a server action and a POST API route caused drift and untested branches.

**Target pattern (implemented):**

```
<form method="post" noValidate onSubmit={bindFormSubmit(...)}>
  → RHF validation
  → fetch POST /api/admin/<action> (JSON body includes token)
  → domain function (e.g. acceptAdminInvite)
  → redirect or inline success
```

`bindFormSubmit` calls `preventDefault` so password managers and native submit cannot leak fields into the query string.

### Credential leak in URL

**Root cause:** `<form>` without `method="post"` and without `preventDefault` performs a **GET** submit. Profile and password fields appeared in the URL; the invite `token` was dropped → false “link expired”.

**Fix stack:**

1. **Primary:** `bindFormSubmit` + POST JSON (same as login/set-password).
2. **Safety net:** Server Component redirect when query contains leaked field names (`password`, `firstName`, etc.), preserving `token` if present.
3. **Proof:** E2E asserts `password=` never appears in URL after submit.

Do not stack sessionStorage / URL scrub / inline scripts as substitutes for fixing native submit.

### Session cookie surprises

| Symptom | Cause | Fix |
| --- | --- | --- |
| `/login` redirects straight to admin landing | Valid `places_agent_session` cookie already set | Use **`GET /login/fresh`** before login E2E or when operator wants a clean sign-in form |
| `500` on `/login?fresh=1` | `cookies().delete()` in a Server Component | Move session clear to a **Route Handler** (`app/login/fresh/route.ts`) then `redirect("/login")` |

### CSRF and LAN dev

Mutating `/api/admin/*` routes check Origin. For cross-machine dev, allow the host LAN IP via env-driven config (see §3). Unit-test LAN origin acceptance when host matches.

---

## 2. Next.js App Router constraints (Next 16)

Evidence from `places-agent` custom server + admin UI.

| Constraint | Symptom | Guidance |
| --- | --- | --- |
| Top-level `await` in CJS | `tsx server.ts` fails: “Top-level await not supported with cjs output” | Wrap startup in async `main()`; no top-level await unless `"type": "module"` |
| Edge middleware + `node:crypto` | `/admin` fails to compile | Cookie **names** in edge-safe module; HMAC/session verify in Node route handlers only |
| `middleware` deprecation warning | Console noise | Matcher still works; do not import Node APIs in middleware/proxy |
| `eslint-config-next` vs TypeScript 7 | `npm run lint` crashes | Run `tsc --noEmit` until typescript-eslint supports TS 7; or pin TypeScript 5.x |
| Turbopack / duplicate dev processes | Random 500s, stale bundles, port conflicts | `make down`, kill stray `tsx watch`, optionally `rm -rf .next`, single `make dev` or `make up` |
| Prisma CLI env | Migrate/seed ignore `.env.local` | Pass `DATABASE_URL=...` on CLI; document in Makefile `db` target |

Custom server (`server.ts`) hosts `/mcp` and `/sse` on the same process as the admin HTML — one Playwright server for E2E.

---

## 3. Cross-machine local dev

Valid operator workflow: dev server on host A, open invite/reset mail on phone or laptop B.

| Setting | Purpose |
| --- | --- |
| `PUBLIC_BASE_URL=http://<host-lan-ip>:<PORT>` | Absolute links in mail point at a host B can reach |
| `ALLOWED_DEV_ORIGINS=<host-lan-ip>` | Next dev serves JS/HMR when Origin is the LAN IP (comma-separated) |
| Listen on `0.0.0.0` | Required for LAN reachability (Docker default; verify for local `make dev`) |

When `PUBLIC_BASE_URL` is unset, URL builders default to `http://localhost:${PORT}` — wrong for device B unless the operator manually edits the link.

**DoD gate:** Operator confirms invite accept on their real cross-machine path, not only localhost E2E.

---

## 4. Playwright — operator admin E2E

Admin UI is a **dynamic** app. Start the real server, then automate (Python `playwright.sync_api`, Chromium headless).

### Harness

- Run `make db` (migrate + seed) before E2E so default admin exists.
- One process: admin HTML + `/api/admin/*` + HTTP tools.
- Prefer **`data-testid`** and roles over English copy (i18n keys are the product contract).
- **`networkidle` is unreliable** against `next dev` (HMR). Use `domcontentloaded` + wait on test ids.
- Clear session with **`/login/fresh`** before login journeys.
- **`page.expect_popup()`** for `target=_blank` links (e.g. instructions from home).
- Smoke **mobile viewport** on changed screens.
- Verify TCP port is free — a stale process on `PORT` makes E2E hit the wrong app.

### Minimum journeys (MVP-1)

| Journey | Proves |
| --- | --- |
| Invite token → accept form → `?done=1` → login → landing | Feature 15 US4; no credentials in URL |
| Home → instructions popup | Features 14, 18; agent id literal |
| Login fail + register disabled | Feature 15 failure |
| Issue caller key → HTTP tool call with secret | Features 17, 12 |
| Locale EN → HK | Feature 19 smoke |

Tool/search behavior belongs in **Vitest HTTP/MCP contract**, not admin UI E2E.

### Gaps (accepted with warnings at MVP-1 close)

- Delete-admin dialog flow: unit/API covered; no Playwright journey yet.
- Four-locale E2E: only EN→HK smoke; CN/TW via unit catalog tests.

---

## 5. Admin mail and destructive mutations

### Resend and dev outbox

- Production: `RESEND_API_KEY` required; failed send → mutation fails (e.g. delete admin AC4).
- Dev without key: in-memory **outbox** (`getMailOutbox()` for tests).
- Dev with key but Resend error: **non-production** falls back to outbox so local flows stay testable (`e2e-invite@example.com` pattern).

Mail bodies use **i18n keys**; absolute URLs from `PUBLIC_BASE_URL` / `APP_URL`.

### Delete admin (pattern for other destructive actions)

- Confirm in UI dialog; disable button during request; show error inside dialog.
- Guards: no self-delete, no last admin — hide UI and enforce in API.
- Notification mail is **informational** (no action token / CTA URL).

---

## 6. i18n in admin web apps

- All user-visible strings through locale catalogs (`EN`, `CN`, `HK`, `TW`).
- Tests assert **keys**, test ids, or role — not a single language’s sentence.
- Protocol ids (`places-agent`, vendor codes) stay **literal**, not translated.
- HK and TW are separate catalogs; fallback is locale → `EN` → key, never HK↔TW swap.

---

## 7. Two auth modes — test separately

| Mode | Used for | Must not |
| --- | --- | --- |
| Admin session cookie | Operator UI, `/api/admin/*` | Authorize HTTP/MCP tools |
| Caller API key (`Authorization: Bearer`) | `/v1/*` tools | Open admin landing |

E2E and contract tests must keep these boundaries explicit.

---

## 8. Definition of Done — web app slice

Checklist distilled from MVP-1 close (extends workspace `dod.mdc` + `common-test-strategy`):

| Gate | MVP-1 status |
| --- | --- |
| Vitest unit + contract green | ✅ 69 tests |
| `tsc --noEmit` | ✅ |
| Playwright critical admin journeys | ✅ |
| User confirms usable on real path | ✅ accepted 2026-08-18 |
| Lint clean | ⚠️ blocked by TS 7 / eslint |
| Coverage ≥ 80% measured | ⚠️ no coverage script yet |
| Git commit / push | ⚠️ pending operator |
| Spec status reflects acceptance | ✅ `2.agent-ac.md` |

**Anti-patterns (do not repeat):**

- Ship token forms without E2E submit + URL leak assert.
- Call invite/auth done after `curl` only.
- Verify only HTTP **or** only MCP for dual-channel tools.
- Mock admin flows that only work against fake mail.

---

## Links

- Auth refactor decision log: `1.places-agent/agent-specs/7.auth-refactor-plan.md`
- Test strategy: `1.places-agent/agent-specs/4.test-strategy.md`
- Visual chrome family: [`../ui/agent-mate-admin-visual.md`](../ui/agent-mate-admin-visual.md)
- MVP slice: [`../../adr/ADR-018-mvp-by-capability.md`](../../adr/ADR-018-mvp-by-capability.md)
