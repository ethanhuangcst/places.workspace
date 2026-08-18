---
title: places-agent Next.js 16 local runtime
type: ops-lesson
status: superseded
as_of: 2026-08-17
superseded_by: ../web-app-development/lessons-from-places-agent-mvp1.md
tags:
  - places-agent
  - next
  - mcp
  - e2e
related_spec: 1.places-agent/agent-specs/5.agent-design.md
related:
  - adr/ADR-016-custom-http-server.md
  - ../web-app-development/lessons-from-places-agent-mvp1.md
---

# places-agent Next.js 16 local runtime

**Superseded.** Content merged into [`../web-app-development/lessons-from-places-agent-mvp1.md`](../web-app-development/lessons-from-places-agent-mvp1.md) (§ Next.js App Router constraints, § Playwright).

Quick pointer:

- Custom server: async `main()`, no top-level await in CJS
- Edge middleware: cookie names only; HMAC in Node handlers
- Playwright: prefer test ids over `networkidle` on `next dev`
- Prisma CLI: pass `DATABASE_URL` explicitly

See also [ADR-016](../../adr/ADR-016-custom-http-server.md).
