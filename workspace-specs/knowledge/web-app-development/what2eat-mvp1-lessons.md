---
title: what2eat MVP-1 DoD lessons
type: ops-lesson
status: active
as_of: 2026-08-19
tags:
  - what2eat
  - testing
  - nextjs
related_spec: 2.what2eat/2eat-specs/2eat-test-plan.md
---

# what2eat MVP-1 — DoD closure lessons

## Summary

MVP-1 (account & profile) reached DoD after adding BFF contract tests, expanded Playwright gates, coverage on auth/profile APIs, CI, and build hardening. Main pitfalls were `NODE_ENV=development` during `next build`, missing `server-only` mock in Vitest, and E2E reset-token seeding without `DATABASE_URL`.

## Evidence

- **Build:** `/_global-error` prerender fails when `NODE_ENV=development` is set during `next build` (Next.js 16 + React 19). Guard with `make build` → `NODE_ENV=production npm run build`; do not set `NODE_ENV` in `.env*` (use `APP_ENV` instead).
- **Contract tests:** Import App Router handlers directly; mock `next/headers` cookies and `server-only` in `tests/setup.ts`. Use isolated `what2eat_test` DB via `TEST_DATABASE_URL`.
- **E2E:** Chain `test_mvp1.py`, register errors, failed login, reset/set-password in `e2e/run.py mvp1`. Seed reset tokens via `e2e/seed_reset_token.ts` with explicit `DATABASE_URL` (dev DB).
- **Coverage:** Scope thresholds to MVP-1 paths (`src/auth`, `app/api/auth|profile|locale`) — not whole repo including MVP-2 Decide UI.

## Lesson / guidance

| Area | Do |
| --- | --- |
| Build / CI | `make build` always forces production NODE_ENV; remove NODE_ENV from env templates |
| Vitest + Next | Mock `server-only`; `fileParallelism: false` with shared Postgres test DB |
| E2E mail reset | Dev outbox in `mail.ts` for contract tests; E2E uses DB token seed script |
| DoD gate | `make quality` = lint + test-coverage + build + test-e2e-mvp1 |

## Links

- [what2eat 测试计划](../../2.what2eat/2eat-specs/2eat-test-plan.md)
- [lessons-from-places-agent-mvp1.md](./lessons-from-places-agent-mvp1.md)
