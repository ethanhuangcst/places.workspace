---
title: places-agent local daemon — make dev vs make up
type: ops-lesson
status: active
as_of: 2026-08-19
tags:
  - places-agent
  - next
  - macos
  - devops
related_spec: 1.places-agent/agent-specs/6.agent-deployment.md
related:
  - ../web-app-development/lessons-from-places-agent-mvp1.md
  - ../../adr/ADR-016-custom-http-server.md
---

# places-agent local daemon — make dev vs make up

## Summary

Once running, places-agent is stable under load (health burst, HTTP contract tests). **Starting and keeping** the process alive on **macOS** depends on how it is launched. `make dev` in a dedicated terminal is reliable; `make up` from short-lived shells (including IDE agents) often reports health OK then the process exits.

## Evidence

- Stability review (2026-08-19): same PID for 90s; 50 sequential + 20 parallel `/v1/health` OK; 15/15 `http-tc-h.test.ts` pass while server stayed up.
- `make up` passes health, then port :3010 is unreachable within seconds when the launcher shell exits.
- macOS lacks `setsid` (util-linux); `scripts/dev-up.sh` falls back to `nohup`.
- `.data/server.log` showed many `places-agent listening` lines without fatal errors — repeated start/stop cycles, not mid-run crashes.
- Foreground `exec npx tsx server.ts` in a persistent background terminal remained healthy.

## Lesson / guidance

| Goal | Command | Notes |
| --- | --- | --- |
| Daily local dev | `make dev` | Foreground in a terminal you keep open |
| Check status | `make status` | Polls `.data/server.pid` + `/v1/health` |
| Clean stale Next lock | `make reset-dev` | After ENOENT / dev lock issues |
| Stop | `make down` | Kills listener on `PORT`, clears `.next/dev/lock` |

**macOS / IDE agents:** treat `make up` as **best-effort**. Do not assume the server survives after a one-shot agent command. For Cursor-driven workflows, start the server in a **long-lived terminal** (background shell with `make dev` or `exec npx tsx --env-file=.env.local server.ts`).

**Linux / CI:** `setsid` path in `dev-up.sh` should detach correctly; verify with `make up && sleep 30 && make status`.

**Production:** use container restart policy or a process manager with `NODE_ENV=production` and a built Next artifact — not Next dev + `nohup`.

## Graceful shutdown (MVP-3a, 2026-08-20)

`server.ts` now registers `SIGTERM` and `SIGINT` handlers:

1. `httpServer.close()` — stop accepting new connections
2. `mcpSessions.close()` + `sseSessions.close()` — clear all managed sessions
3. `process.exit(0)` on clean close; `setTimeout(() => process.exit(1), 10_000).unref()` as force kill

`dev-server.sh` exports `NODE_ENV=development` before `tsx` to prevent `.env.production` pollution (Safari Secure cookie bug — see [`safari-secure-cookie-localhost.md`](./safari-secure-cookie-localhost.md)).

## Follow-up (optional)

- Harden `dev-up.sh` with double-fork + `disown` on macOS, or document `make up` as Linux-only.
- Wire `make status` to refresh PID from `lsof` when `.data/server.pid` is stale.

## Links

- Makefile targets: `1.places-agent/Makefile`
- Custom server: [ADR-016](../../adr/ADR-016-custom-http-server.md)
- Next runtime notes: [`../web-app-development/lessons-from-places-agent-mvp1.md`](../web-app-development/lessons-from-places-agent-mvp1.md) §2
