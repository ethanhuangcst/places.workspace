---
title: what2eat MVP-4 post-close follow-ups
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - what2eat
  - places-agent
  - mvp-4
  - chat
  - providers
related_spec: 2.what2eat/2eat-specs/2eat-stories.md
related:
  - knowledge/web-app-development/what2eat-mvp4-lessons.md
  - knowledge/web-app-development/what2eat-chat-provider-auto-select.md
  - knowledge/web-app-development/what2eat-chat-agent-timeout.md
  - knowledge/web-app-development/what2eat-decide-locale-draft.md
  - adr/ADR-026-region-based-provider-auto-selection.md
  - adr/ADR-029-decide-criteria-draft-hydrate.md
  - adr/ADR-031-amap-empty-google-fallback.md
---

# what2eat MVP-4 — post-close follow-ups

Lessons from bugs found while exercising MVP-4 chat against live Shanghai pins (2026-08-20 evening).

## Process assets (what changed)

| Area | Change |
| --- | --- |
| SSR / draft | `decideFormSsrDefaults` + mount hydrate; panel size same pattern |
| List chat key | Stable `w2e.chat.list` (not per-`searchId`); migrate legacy keys |
| Chat timeout | `chatTimeoutMs()` default ≥ 90s (`PLACES_AGENT_CHAT_TIMEOUT_MS`) |
| Provider parity | Chat strips `providers` on search/geocode; region uses `address ?? query` |
| Empty AMAP | ADR-031: auto AMAP-empty → one Google pass |
| Card-first | Prompt + agent `places[]` + BFF `picksFromAgentPlaces` / `mergeHydratePicks` |
| Block parse | Accept nested `{heading:{…}}` and `{type:"heading",…}` |
| Local daemons | Agent/`make up` + what2eat `nohup`/`make up` flaky on macOS agent shells |

## Failure → fix map

| Symptom | Root cause | Fix |
| --- | --- | --- |
| Hydration mismatch on Decide chat context | `useState(() => sessionStorage…)` vs SSR defaults | SSR defaults; apply draft after mount |
| Chat history wiped on re-search | Key `w2e.chat.list.{searchId}` | Stable `w2e.chat.list` |
| 「暂时无法连线 agent」 | BFF 25s abort; agent healthy | Separate chat timeout ≥ 90s |
| Chat used Google for 吴中路 | LLM passed `providers:["GOOGLE_MAPS"]`; query ignored for region | Strip providers; `address ?? query` |
| 「吴记鲜定位」找不到 | Typo 定位≠定味; mainland AMAP-only empty | ADR-031 Google empty fallback |
| Long text, no cards | Model text-dumps; hydrate only SearchCache | Card-first prompt; return/merge `places[]` |

## Operator notes

1. After agent code changes: `cd 1.places-agent && make down && make up` (verify `make status` after ~10s).
2. what2eat: prefer a **persistent** Cursor/`npm run dev` terminal; short-lived `nohup` from agent shells may die.
3. Do not edit `.env.local` without confirmation (`protect-eng`); chat timeout defaults in code if unset.

## DoD note

MVP-4 stories are implemented and automated gates were green; **product DoD still needs explicit user usability confirm**.
