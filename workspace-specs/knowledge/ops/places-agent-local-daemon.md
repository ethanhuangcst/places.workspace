---
title: places-agent local daemon — make dev vs make up
type: ops-lesson
status: active
as_of: 2026-08-22
tags:
  - places-agent
  - next
  - macos
  - devops
  - what2eat
related_spec: 1.places-agent/agent-specs/6.agent-deployment.md
related:
  - ../web-app-development/lessons-from-places-agent-mvp1.md
  - ../web-app-development/what2eat-mvp4-followups.md
  - ../../adr/ADR-016-custom-http-server.md
  - ../../adr/ADR-035-macos-agent-daemon-detach.md
---

# places-agent local daemon — make dev vs make up

## Summary

Once running, places-agent is stable under load (health burst, HTTP contract tests). **Starting and keeping** the process alive on **macOS** depends on how it is launched. `make dev` in a dedicated terminal is reliable; `make up` from short-lived shells (including IDE agents) often reports health OK then the process exits.

**2026-08-22：** 确认 Darwin 无 `setsid` 时 `nohup &` 仍会在 Cursor agent shell 结束后丢掉 :3010；用 Python `start_new_session=True` 拉起后 health 可持续。判定「已启动」必须以 LISTEN+`/v1/health` 为准，勿信 stale `.data/server.pid`。

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

**what2eat (`:3020`):** same class of failure — `nohup npm run dev` from a short-lived agent shell often dies after minutes. Prefer a persistent Cursor background terminal (`npm run dev`) or verify with `curl :3020` after ~10s. Postgres `make up` is separate from the Next process.

**Linux / CI:** `setsid` path in `dev-up.sh` should detach correctly; verify with `make up && sleep 30 && make status`.

**Production:** use container restart policy or a process manager with `NODE_ENV=production` and a built Next artifact — not Next dev + `nohup`.

## Graceful shutdown (MVP-3a, 2026-08-20)

`server.ts` now registers `SIGTERM` and `SIGINT` handlers:

1. `httpServer.close()` — stop accepting new connections
2. `mcpSessions.close()` + `sseSessions.close()` — clear all managed sessions
3. `process.exit(0)` on clean close; `setTimeout(() => process.exit(1), 10_000).unref()` as force kill

`dev-server.sh` exports `NODE_ENV=development` before `tsx` to prevent `.env.production` pollution (Safari Secure cookie bug — see [`safari-secure-cookie-localhost.md`](./safari-secure-cookie-localhost.md)).

## Refresh 2026-08-22 (Cursor agent + :3010)

| Symptom | Cause | Fix |
| --- | --- | --- |
| health OK → seconds later ECONNREFUSED | `dev-up.sh` falls back to `nohup`; agent shell reaps process group | Detach with `start_new_session=True` / `os.setsid` (ADR-035) |
| `make status` / pid file says up | Stale `.data/server.pid` + `.next/dev/lock` | Require LISTEN(PORT) + `/v1/health`; clear pid/lock on fail |
| Many `tsx watch server.ts` with no :3010 | Parallel `npm run dev` / watch left orphans | Narrow `pkill` to this repo path; prefer one launcher |

Verified: Python `subprocess.Popen(..., start_new_session=True)` + `NODE_ENV=development` kept :3010 healthy after agent command returned.

## Follow-up

- [x] Implement ADR-035 in `scripts/daemon_detach.py` + thin `dev-up.sh` / `dev-down.sh` (2026-08-22).
- [x] `make status` uses LISTEN + `/v1/health` via `daemon_detach.py status`.
- [x] Unit tests: `make test-scripts`.
- [x] ADR-043 D9 (P0 must_include 强制排日 + P1 末日卡单次展示) — restart daemon after D9 edits so ChatBox hits the new build (2026-08-23). Old ChatBox conversations that hit the pre-D9 build (e.g. `Fix candidates` / D7-only paths) cannot serve as D9 regression evidence — start a fresh trip in ChatBox after `make down && make up` (or `python3 scripts/daemon_detach.py start` when Docker/Postgres is unavailable).
- [x] ADR-043 D9 follow-up — `buildDayCardMarkdown` no longer leaks internal reason markers (`hard_must_include` / `hard_must_see`) into `user_visible_markdown`; multi-day `covered` stickiness and no-reinject locked in by tests (2026-08-23). Restart daemon after this edit.
- [x] ADR-043 D9 精简 (C3 de-scope) — 删除 P1 展示死代码、`must_include` assignment 降级、`ensureHardMustIncludeCoverage` 注入改为硬失败重试、删除五处城市硬编码（`must-see-coverage` / `discover-must-see` CATALOG / `discover-dedupe` 城市分支 / `place-filters` 城市后缀正则 / `FAR_DISTRICT_HINT`），新增 `discover-must-see-llm` LLM 推断 + `no-city-hardcode-guard` 守卫测试 (2026-08-23)。**改完必须 `python3 scripts/daemon_detach.py down && sleep 1 && python3 scripts/daemon_detach.py start` 重启 daemon**，否则 ChatBox 仍命中旧 build，C3 行为无法端到端验证（注意 `restart` 不是有效 action，需显式 `down` 再 `start`）。端到端验证：Lisbon `discover_places` 返回 `inferred_must_see=['贝伦塔','热罗尼莫斯修道院','圣若热城堡']`；`arrange_day` 传 `must_include=['辛特拉']` 后 focus=辛特拉、covered=['辛特拉']，用户显式 must_include 优先于 inferred must-see。
- [x] ADR-043 D9 精简 follow-up (theme 门控 + 全天 prompt + 续排措辞) — `selectMustIncludeFocusToken` 改 theme 门控（仅当 `day_theme` 命名 missing token 才强制 focus；无 theme → null，让 themed 那天认领，末日闸兜底）；`buildSchedulePrompt` 去掉「half-day」鼓励、改为 day-trip 小镇 dedicate full day；续排措辞经三轮实验定稿 step2「ONE day at a time, no asking」（实验记录见 ADR-043 D9 精简 item 8）(2026-08-23)。重启 daemon 同上。**已知限制：续排顺序（不并发、不问「如果你愿意」）依赖宿主 LLM 遵守 host_instructions，服务端无硬保证——软闸对同回合并发有竞态，且挡不住「问完再发」。**

## Links

- Makefile targets: `1.places-agent/Makefile`
- Custom server: [ADR-016](../../adr/ADR-016-custom-http-server.md)
- macOS detach decision: [ADR-035](../../adr/ADR-035-macos-agent-daemon-detach.md)
- Implementation: `1.places-agent/scripts/daemon_detach.py`
- Next runtime notes: [`../web-app-development/lessons-from-places-agent-mvp1.md`](../web-app-development/lessons-from-places-agent-mvp1.md) §2
