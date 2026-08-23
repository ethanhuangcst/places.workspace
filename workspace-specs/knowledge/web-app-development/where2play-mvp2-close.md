---
title: where2play MVP-2 close (save loop + live E2E)
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - where2play
  - mvp2
  - e2e
  - openai-cn
related:
  - adr/ADR-037-where2play-plan-l2-quanzil.md
  - knowledge/llm/openai-cn-gateway.md
  - knowledge/web-app-development/what2eat-mvp2-lessons.md
---

# where2play MVP-2 close

## Summary

MVP-2 merged Plan progressive + interest prefill + save/saved/unsave into one slice. Live gate: `make test-e2e-mvp2-live` (London, 1 day). A common live failure was **arrange L2** returning `errors.provider_failed` when `OPENAI_BASE_URL` omitted the `/v1` suffix — the gateway returned HTML and the BFF treated it as provider failure.

## Evidence

- `POST /api/plan` NDJSON: `discover_done` succeeded (8+8 candidates) then `error` in ~3s — not a discover/agent timeout.
- Direct probe: `https://happycodeai.com/chat/completions` → HTML 200; `https://happycodeai.com/v1/chat/completions` → JSON.
- Fix: `openaiApiBaseUrl()` in `src/core/openai-config.ts` appends `/v1` when env is host-only.
- E2E journey (~29s): register → interest prefill → plan → save → saved grid → detail → unsave → empty.

## Lesson / guidance

| Check | Action |
| --- | --- |
| Live plan fails fast after discover | Probe `${OPENAI_BASE_URL}/chat/completions` (or normalized `/v1` path) returns JSON, not HTML |
| E2E `plan-save` timeout | Usually upstream plan error, not save UI — read `plan-error` / NDJSON `error` key first |
| Operator env | Prefer full `OPENAI_BASE_URL=https://…/v1` per `OPENAI_CN-gateway.md`; code normalizes host-only as safety net |
| Probe destination | London EN locale; `PLACES_AGENT_*` local on `:3010`; Postgres `:5435` |

## Links

- Test plan: `3.where2play/2play-specs/2play-test-plan.md` — `make test-e2e-mvp2-live`
- Stories: features 14–22, 30; `plan-07` AC1 Done
- **Follow-on:** **MVP-3** Mode H + 真交通 + 地标 — see `where2play-plan-l2-quanzil.md`
