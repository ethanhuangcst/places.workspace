---
title: places family on 野草云3 — release deltas vs kb-agent
type: ops-lesson
status: active
as_of: 2026-08-17
tags:
  - ops
  - release-bot
  - places-agent
  - 野草云3
related_spec: workspace-specs/6.deployment-plan.md
related:
  - knowledge/handbook.md
  - knowledge/ops/places-agent-next-runtime.md
  - adr/ADR-009-deploy-option-1.md
  - adr/ADR-012-admin-ui-on-agent.md
  - adr/ADR-015-sqlite-prisma.md
  - adr/ADR-016-custom-http-server.md
---

# places family on 野草云3 — do not copy kb-agent NPM blindly

## Summary

Release-bot’s generic playbook (GHCR → Portainer pull-only → NPM → Cloudflare on 野草云3 / `38.55.192.140`) still applies. places-agent is **not** kb-agent: one Node process, one container, SQLite on a volume, no Custom Locations to a second upstream. Full tables: [`6.deployment-plan.md`](../../6.deployment-plan.md).

## Evidence

- ADR-009: three stacks (`places-agent`, `what2eat`, `where2play`). ADR-012: operator UI is inside the agent image, not a fourth stack.
- ADR-016: `CMD` is `node server.ts`. `/mcp` `/sse` `/messages` share the Next process. kb-agent splits `kb-web` and `kb-agent`, so NPM Custom Locations route `/mcp` to a second container — that pattern is **wrong** here.
- ADR-015: `DATABASE_URL=file:/data/places-agent.db` on volume `places_agent_data`. Do not put this stack on Aliyun Postgres `101.132.156.250` (that host is `kb_agent` / `mypoke_trade_prod` / `media_marketing`).
- ADR-017: Google Maps Worker MCP is a **Cloudflare Worker**, not a Portainer service.
- Inventory as_of 2026-08-12 (release-bot `svr_hk_vps_3/hk_vps_3_setting.md`): taken/reserved `3001`–`3003`, `3006`, `3200`–`3203`, `6333`, `6335`, `6336`. **Proposed** host ports (confirm with `ss` before first deploy): what2eat `3004`, where2play `3005`, places-agent `3007`. NPM Forward Port is always container **3000**.
- release-bot ADR-002: `IMAGE_TAG` is a published GHCR tag, never git branch `main`. ADR-003: after stack recreate, **Save** the NPM host even if fields are unchanged; homepage 200 ≠ MCP healthy.

## Lesson / guidance

| Do | Do not |
| --- | --- |
| One hostname `places.agent-mate.ai` → container `places-agent:3000` for `/`, `/v1`, `/mcp`, `/sse`, `/messages` | kb-style Custom Locations to a second container |
| SSE: `proxy_buffering off`, timeouts ≥ 300s; Websockets on | Treat homepage 200 as MCP healthy |
| Cursor: `https://places.agent-mate.ai/mcp` + Bearer. ChatBox: `…/sse` (not `/mcp`) | Point ChatBox at `/mcp` |
| Caller-visible id `places-agent` (ADR-013). Health `{ "agent": "places-agent", "ok": true }` | Use the hostname as the agent id |
| SQLite migrate + seed on boot; backup volume before schema migrate | Expect `IMAGE_TAG` rollback to restore `/data` |
| Seed `admin` / `me@ethanhuang.com`; public register off | Bake a password into the image; set `DEV_ADMIN_PASSWORD` in Portainer |
| First wave = places-agent (MVP-1). Apps later | Wait for what2eat/where2play Docker before using the family plan |
| Never recreate `portainer_network`; spot-check ≥1 existing app | Build images on the VPS |

**App-repo blockers (places-agent) before Portainer:** `server.ts`, `Dockerfile` (`CMD node server.ts`), `docker-compose.prod.yml` (image-only, external `portainer_network`), `.github/workflows/ghcr.yml`, `.env.prod.example`. Prisma seed today reads `DEV_ADMIN_PASSWORD` only — prod first boot is empty hash → `/set-password` until seed is wired.

**Env files:** do not rewrite `.env` / `.env.local` / `.env.example` or delete user-entered keys without explicit confirmation (personal rule `protect-eng`). Specs list **env names only**.

## Links

- [`6.deployment-plan.md`](../../6.deployment-plan.md)
- [ADR-009](../../adr/ADR-009-deploy-option-1.md), [ADR-012](../../adr/ADR-012-admin-ui-on-agent.md), [ADR-015](../../adr/ADR-015-sqlite-prisma.md), [ADR-016](../../adr/ADR-016-custom-http-server.md)
- release-bot `knowledge/03-semi-auto-release.md`, `knowledge/09-isolation-safety.md`
