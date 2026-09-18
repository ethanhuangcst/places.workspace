# MVP-T9 close lessons (chat 改行程)

**Date:** 2026-09-18

## Delivered

1. **agent-chat-93e** — `plan_trip` refine mode (`refine.instruction` + `trip_id`); `commit_trip.operations[]` patch skeleton; returns `reply` + `itinerary`.
2. **2play-plan-050** — Removed BFF product LLM (`plan-arrange-llm`, `chat-assistant`, `llm-chat-config`); legacy `plan-day-by-day` path dropped from `/api/plan`.
3. **2play-plan-90e** — `/api/chat` forwards to agent refine; `plan-nav-input` composer after `planCompleteLine`; `w2p.chat.draft` localStorage.

## Operator note (protect-eng)

After deploy, you may remove unused where2play `.env` keys: `OPENAI_*`, `QWEN_*`, `PLAN_ARRANGE_*` — confirm before editing env files.

## Related

- [plan-trip-refine-t9.md](../agent/plan-trip-refine-t9.md)
- ADR-050
