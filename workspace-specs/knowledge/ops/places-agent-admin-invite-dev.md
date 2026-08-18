---
title: places-agent admin invite — dev testing lessons
updated: 2026-08-18
status: superseded
superseded_by: ../web-app-development/lessons-from-places-agent-mvp1.md
---

# Admin invite accept — development and testing

**Superseded.** Content merged into [`../web-app-development/lessons-from-places-agent-mvp1.md`](../web-app-development/lessons-from-places-agent-mvp1.md) (§ Cross-machine local dev, § Auth forms).

Quick pointer:

- `PUBLIC_BASE_URL` + `ALLOWED_DEV_ORIGINS` for cross-machine invite testing
- POST-not-GET via `bindFormSubmit`; E2E asserts no `password=` in URL
- Refactor plan: `1.places-agent/agent-specs/7.auth-refactor-plan.md`
