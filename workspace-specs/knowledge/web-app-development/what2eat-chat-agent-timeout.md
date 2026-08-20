---
title: what2eat chat agent timeout
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - what2eat
  - chat
  - timeout
related_spec: 2.what2eat/2eat-specs/2eat-stories.md
related:
  - knowledge/web-app-development/what2eat-mvp3-lessons.md
---

# Chat “无法连线 agent” — BFF timeout

## Summary

UI copy `errors.chat_failed` (“暂时无法连线 agent”) appears when `POST /api/chat` returns non-OK. Live Next logs showed **`POST /api/chat 502 in 25.1s`** — the BFF AbortController fired at `PLACES_AGENT_TIMEOUT_MS` (25000), not a dead agent. Agent `/v1/health` was 200; many chat turns succeed in 7–24s; tool-heavy turns (e.g. user says “上海市” → geocode + search in the agent loop) need longer.

## Evidence

- what2eat `.env.local`: `PLACES_AGENT_TIMEOUT_MS=25000`
- `chat()` used the general timeout; `searchRestaurants()` already had a 60s budget
- Successful chats: 7–24s; failures clustered at **25.0–25.1s** → abort

## Lesson / guidance

1. Use `chatTimeoutMs()` / `PLACES_AGENT_CHAT_TIMEOUT_MS` (default **≥ 90s**) for `/v1/chat`.
2. Do not treat `errors.chat_failed` as “agent down” until health fails or logs show connection refused.
3. Long list-chat transcripts increase LLM latency; still keep the chat budget above one tool round-trip + model time.

## Links

- `2.what2eat/src/places-agent/client.ts` — `chatTimeoutMs`
- Deployment: `PLACES_AGENT_CHAT_TIMEOUT_MS` in `2eat-deployment-plan.md`
