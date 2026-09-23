# Bug list — E2E regression 2026-09-23

**Run date:** 2026-09-23  
**Stack:** places-agent `:3010`, what2eat `:3020`, where2play `:3030`, Postgres `:5435`  
**Scope:** Full Makefile Playwright e2e for what2eat + where2play (parity = Lisbon / Rome / Prague subset).  
**Policy:** Record only; no product fixes in this cycle (except stack start / DB migrate / correct `DATABASE_URL` for app processes).

## Environment notes (not product bugs)

| Note | Detail |
| --- | --- |
| Live servers reused | `:3010` / `:3020` / `:3030` already healthy; apps had no process-env `DATABASE_URL` (use `.env.local`). Parent shell had `where2play` URL — e2e scripts set per-app `DATABASE_URL` / `W2*_BASE_URL`. |
| `make test-e2e-mvp1` avoided | Driven as `python3 e2e/*.py` against live servers to avoid `with_server` second `npm run dev` (`EADDRINUSE`). |
| Unit smoke (fixture) | what2eat + where2play `npm test`: setup `prisma migrate deploy` **P3009** on `*_test` DBs — all files failed to load (env/migrate history, not product). places-agent: **26 failed / 1008 passed** (pre-existing unit failures; not expanded in this e2e cycle). |
| Agent `make test` | `db-up` failed (docker compose `-f`); used `npx vitest run` with `TEST_DATABASE_URL` on `:5435`. |
| Parity INDEX overwrite | Harness rewrites `INDEX.md` per `--only` city; last write is Prague only. Per-city md files `01-lisbon` / `04-rome` / `11-prague` updated OK. |

## Results matrix

### what2eat

| Test | Result |
| --- | --- |
| `test_mvp1.py` | PASS |
| `test_register_errors.py` | PASS |
| `test_login_failed.py` | PASS |
| `test_reset_set_password.py` | PASS |
| `test_mvp2_live.py` | PASS |
| `test_mvp3_live.py` | PASS |
| `test_mvp4_live.py` | PASS |

Pages exercised by PASS paths: `/`, `/register`, `/login`, `/profile`, `/decide`, place dialog, `/saved`, history, reset/set-password.

### where2play (Makefile targets)

| Test | Result |
| --- | --- |
| `test_mvp1.py` | PASS |
| `test_register_errors.py` | PASS |
| `test_login_failed.py` | PASS |
| `test_reset_set_password.py` | PASS |
| `test_mvp2_live.py` | FAIL — BUG-007 |
| `e2e_t3_skeleton_hz.py` | PASS |
| `probe_t3_ui_verify.py` | PASS |
| `e2e_draft_persist_hz.py` | PASS |
| `test_mvp3_live.py` | FAIL — BUG-007 |
| `test_mvp10_structure.py` | PASS |
| `probe_plan_lisbon.py` | PASS (exit 0); soft note: printed `AC3 first block within 09:30±5min: FAIL` |
| `test_agent_parity_30.py` (`--only lisbon,rome,prague`) | PASS — see agent parity |
| `test_mvp11_nationality.py` (`W2P_E2E_NATIONALITY=1`) | PASS |

### Agent parity (Lisbon=1, Rome=4, Prague=11)

Harness `places-agent/scripts/e2e-places-agent.py` via HTTP `/v1` (delegated from `where2play/e2e/test_agent_parity_30.py`):

| # | City | Result | Duration | Artifact |
| --- | --- | --- | --- | --- |
| 1 | Lisbon | PASS (`trip_complete`, 24 calls) | 195.0s | `01-lisbon.md` |
| 4 | Rome | PASS (22 calls) | 181.7s | `04-rome.md` |
| 11 | Prague | PASS (17 calls) | 121.1s | `11-prague.md` |

---

## Open bugs

| id | product | page / area | test | severity | expected | actual | repro | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| BUG-001 | what2eat | `/profile` likes | `test_mvp1.py` | medium | After save + re-login, likes value holds cuisine keys | E2E asserted EN copy | Fixed 2026-09-22 | fixed |
| BUG-002 | where2play | `/register` | most Makefile e2e | high | Register picks ISO nationality | Scripts omitted nationality | Fixed 2026-09-22 | fixed |
| BUG-003 | where2play | e2e tooling | `make test-e2e-chat02` | medium | Dead target after ADR-071 | File missing | Removed 2026-09-22 | removed |
| BUG-004 | where2play | e2e tooling | `test_agent_parity_30.py` | medium | Resolve workspace agent script + city→id | Wrong path | Fixed 2026-09-22 | fixed |
| BUG-005 | where2play | `/profile` nationality | `test_mvp11_nationality.py` | medium | Open list shows all codes | Query filtered to selected label | Fixed 2026-09-22 | fixed |
| BUG-006 | places-agent | HTTP `/v1` parity | `e2e-places-agent.py` | high | Parity over HTTP `/v1` | Empty MCP payload after ADR-076 | Fixed 2026-09-22 | fixed |
| BUG-007 | where2play | `/plan` Hangzhou live fill UI | `test_mvp2_live.py`, `test_mvp3_live.py` | high | After plan completes, `#plan-itinerary .slot` (non-candidate/pending) count &gt; 0 so save/detail journeys can proceed | `plan-save` became enabled (or done wait satisfied) but **zero** `.slot` rows | Register → takeoff Hangzhou → confirm → wait save/done → `slots.count() == 0`. Contrast: `e2e_draft_persist_hz` (1d fill) and `e2e_t3_skeleton_hz` PASS | **open** |

## Coverage gaps (pages without a green Makefile case)

| Product | Page | Notes |
| --- | --- | --- |
| where2play | `/saved/[id]` | Blocked this run by BUG-007 (mvp2-live never reached save/detail) |
| where2play | `/debug/plan` | No Makefile e2e target |
| what2eat | `/debug/plan` (if present) | No Makefile e2e target |

## Suggested follow-ups

1. **BUG-007** — RCA whether hangzhou multi-day fill never lands place slots, or e2e waits on `plan-save` which enables at skeleton/`pagePhase=done` before fill (selector race). Prefer waiting for filled place slots / `plan-thread-complete` before asserting `.slot`.
2. Repair what2eat/where2play `*_test` Prisma migrate history (P3009) so unit smoke is runnable.
3. Triage places-agent 26 failing unit tests separately from this e2e matrix.
4. Soft: Lisbon probe AC3 first-block time window (script still exit 0).
