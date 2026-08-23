---
title: MCP client integration — Cursor and ChatBox
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - places-agent
  - mcp
  - cursor
  - chatbox
related_spec: 1.places-agent/agent-specs/6.agent-deployment.md
related:
  - ../../adr/ADR-003-dual-transport.md
  - ../../adr/ADR-019-http-first-user-test-automation.md
  - ../../adr/ADR-040-plan-itinerary-align-split-tools.md
  - ../ops/places-agent-local-daemon.md
  - ./mcp-client-integration-chatbox-prompt.md
---

# MCP client integration — Cursor and ChatBox

## Summary

places-agent serves MCP on the **same origin** as HTTP tools. Client configuration differs by host: Cursor uses **Streamable HTTP** at `/mcp`; ChatBox uses **Remote HTTP/SSE** at `/sse`. Remote configs use `url` + `headers` — not local stdio `command` / `args`.

**ChatBox product path does not require users to change the assistant system prompt.** One-question intake and day-by-day arrange come from **tool return values** (`intake` + `host_instructions`) and MCP defaults (`arrange_day` omit `execution` → `agent`).

## Evidence

- Routing in `1.places-agent/server.ts`: `/mcp` → Streamable HTTP; `/sse` → GET opens SSE, POST may initialize MCP (ChatBox pattern).
- Operator integration guide: `app/instructions` (`GuideCopyBlock`, i18n keys under `admin.guide.*`).
- Cursor remote MCP docs: `mcpServers.<name>.url` + optional `headers`; env interpolation `${env:VAR}`.
- Manual tests: ChatBox must not point at `/mcp`; header `Authorization: Bearer <caller_api_key>`.
- Remote local testing: `localtunnel --port 3010` exposes HTTPS for ChatBox on another device; first visit may show loca.lt interstitial.
- ADR-040 D3/D4' (2026-08-23): server intake + MCP arrange default `agent` — no client system-prompt gate.

## Lesson / guidance

### Cursor (remote Streamable HTTP)

File: project **`.cursor/mcp.json`** (or Settings → MCP → Edit config).

```json
{
  "mcpServers": {
    "places-agent": {
      "url": "https://places.agent-mate.ai/mcp",
      "headers": {
        "Authorization": "Bearer ${env:PLACES_AGENT_CALLER_KEY}"
      }
    }
  }
}
```

- Set `PLACES_AGENT_CALLER_KEY` in the shell environment, **or** use a literal `Bearer pa_…`.
- Reload Cursor after saving.
- Local dev: `http://localhost:3010/mcp` (match `PORT` in `.env.local`).

### ChatBox (Remote HTTP/SSE)

| Field | Value |
| --- | --- |
| Type | Remote (HTTP/SSE) |
| URL (prod) | `https://places.agent-mate.ai/sse` |
| URL (local) | `http://localhost:3010/sse` |
| Header | `Authorization: Bearer <caller_api_key>` |

Enable MCP tools on the chat. The chat model is ChatBox’s, not places-agent’s OPENAI_CN loop.

Name the connection **`places-agent`**. **Edit MCP Server listing tools ≠ this chat has them.** ChatBox injects tools per conversation. An old thread started before MCP was enabled (or with only kb MCP on) will keep saying `search_restaurants` is missing. Fix: **new chat** → turn on MCP for that chat → enable the `places-agent` server → confirm the composer/tool panel lists `search_restaurants`. Do not ask the model to “self-check” the connection; look at that session’s tool list.

**Multi-MCP is normal (ADR-040 D7).** Do **not** tell operators to disable kb / web-search / other MCP servers. Prefer tool descriptions so the host routes trip asks to places-agent. `GMAPS_MCP_*` remains places-agent’s **server-side** Google fallback (ADR-017), not a ChatBox client tool.

Header field may wrap visually; the value must still be `Authorization=Bearer pa_…` (one logical line). After changing MCP tool descriptions or the server name, Save, Test, then **new chat**.

### Conversational trip path (no client system-prompt change)

1. **Option A (default):** Paste the **fixed 8-row trip form** (city, start, days, optional hotel, pace, spend_level 1–3 节约/适中/宽松 default 2, interests, must-include names) in **one** message. Then `discover_places` **once**. Forbidden: one MCP call per question.
2. Early call → `intake.question` is always the full 8-row form. Defaults: pace=medium, spend_level=2.
3. `arrange_day` day-by-day with **multi-line** day cards; auto-continue. Pass `must_include` every call. Server blocks overview if any must_include uncovered (`present_day_then_cover_must_include`).
4. No hotel → no `from_origin`/`to_destination`; keep between-stop transit. `sources` stay arrays.
5. Do **not** require custom ChatBox system prompts. Optional: [mcp-client-integration-chatbox-prompt.md](./mcp-client-integration-chatbox-prompt.md).

### UI / copy pitfalls

- JSON snippets in the integration guide must render with **`white-space: pre`** (`.code-block--multiline`) or operators see one truncated line and assume wrong format.
- Protocol id **`places-agent`** is literal in MCP `serverInfo.name` and HTTP `agent` — not i18n.

### Open issue — NL trip ask never calls MCP (2026-08-23)

**Status:** mitigated in product path — tool **descriptions** strengthened so city+days / 行程 / N日游 intents prefer `discover_places` (ADR-043 D7). Host routing still non-deterministic; not a ChatBox system-prompt gate.

**Symptom (historical):** In ChatBox (e.g. GPT-4o), short asks like `推荐西安三天行程` / `推荐重庆二日游` sometimes returned parametric Markdown with **no** places-agent tool call.

**Do not confuse with:** must_include hard coverage (D7) — that applies **after** a tool is called.

### must_include hard coverage (ADR-043 D7)

HTTP `/v1/arrange_day` and MCP `arrange_day` share `arrangeDay`:

- Auto-search **one** still-missing must_include token per call; merge into candidates; mark HARD MUST SCHEDULE.
- Covered only if a scheduled block is in that token’s **search pool** (name/`native_id`), or within **10km** of the geocoded anchor **and** name/address matches the token/aliases. **`day_theme` alone never covers.**
- Response includes `must_include_coverage: { must_include, covered, missing }` on both channels.

### Empty candidates auto-discover (ADR-043 D8)

Shared `arrangeDay` (HTTP = MCP):

- After `exclude_names`, if **`places` is empty** and `city` is set → server calls `discoverPlaces` (also fills restaurants when that side is empty). Empty restaurants alone do **not** trigger live discover.
- Missing city + empty pool → clear error (no host invent path).
- Failure `host_instructions` (`ARRANGE_DAY_FAILURE_HOST_INSTRUCTIONS`): retry same dayIndex; do **not** invent POIs / fabricate via `search_places`.
- Prefer passing the `discover_places` pool when the host already has it; empty arrays are still OK.
- Last-day host_instructions: present Day N **ONCE**, overview **ONCE**, then STOP (no re-paste). Day cards: only `blocks[]` from the tool.

### Closed — must_include day-trips not recommended after D7 (2026-08-23 → D9 P0)

**Status:** Closed (ADR-043 D9 P0). Server-side day assignment + deterministic inject + hard fail.

**Symptom:** Post–ADR-043 D7 verification (Lisbon multi-day ChatBox path): user listed must-include **辛特拉 / 卡斯凯什** (Cascais), but the presented itinerary had **no day-trip recommendations** for those places (neither dedicated day nor blocks from the must_include search pool).

**Contrast with prior failure:** Earlier bug was **false coverage** (theme claimed Sintra/Cascais while blocks were Queluz/Belém). This report is **absence** — must-include towns not shown as recommendations at all.

**Root cause (D9):** D7 only added must_include to the candidate pool + prompt pressure (`hard_must_schedule`). `ensureHardMustSeeCoverage` injected Xi'an clusters but not traveler `focusPool`, and the tool returned `ok:true` even when `must_include` remained missing. Host could omit `preferences.must_include` on later calls (no sticky).

**Fix (D9 P0):**
- `must-include-coverage.ts` session gains `assignment: token → dayIndex` (one per day when `numDays >= N`, else round-robin from day 1); `must_include[]` is sticky once seeded.
- `discover_places` and the first `arrange_day` with non-empty `must_include` seed the assignment; later calls that omit `must_include` still recover from the sticky session.
- `prepareMustIncludeFocus` prefers the token assigned to the current `dayIndex` (still only one focus/day).
- New `ensureHardMustIncludeCoverage` (mirrors `ensureHardMustSeeCoverage`) injects the best focus-pool card as `attraction` (`reason: hard_must_include`) when LLM omitted it, then trims soft blocks to pace. No prompt reliance.
- After `applyMustIncludeDayEvidence`, the agent path **hard fails** if the day's focus token is still missing (forbids `ok:true` + missing). Empty focus search → `ok:false`.
- Host (2play Mode H) path returns missing coverage without throwing.

**Verification (P0 hard gates, all green):** 辛特拉/卡斯凯什 across 4 days → both covered in real `blocks[]`; day-2 call omitting `must_include` still focuses the assigned uncovered token (sticky); LLM scheduling only Belém with Sintra in focusPool → injected and covered; focus search empty → `ok:false`; empty `must_include` → no invented day-trip. See `must-include-coverage.test.ts` and `itinerary-planner.test.ts` (D9 P0 block).

**Do not confuse with:** NL trip ask never calling MCP (separate open issue above).

### Mitigated — intake interests examples (2026-08-23)

**Status:** mitigated (D8 copy + richer chips). Host rewrite still possible.

**Server:** interest row includes `历史 / 美食 / 海边 / 街区 / 夜生活 / 街头漫步 / 市集`. `buildIntakeHostInstructions` requires paste `intake.question` **as-is** (do not drop row-7 examples).

### Fixed — arrange_day empty candidates (2026-08-23 → D8)

**Status:** fixed in ADR-043 D8 (`ensureArrangeCandidates` inside `arrangeDay`).

**Was:** empty `places`/`restaurants` → hard fail + “Fix candidates” → host invent/search.  
**Now:** auto-discover from city; error copy no longer pushes host invent.

### Mitigated — last day card duplicated after overview (2026-08-23)

**Status:** mitigated via stronger `present_day_then_overview` host_instructions (ONCE + Forbidden re-paste). Single-message double paste cannot be fully prevented server-side without host compliance.

**Root cause:** host double-wrote Day 4 + overview in one assistant message (no second tool call).

### Known limitation — host_instructions cannot force host LLM tool-call discipline (2026-08-23)

**Status:** mitigated as best-effort via `host_instructions` wording (step2: "ONE day at a time, present Day N then call Day N+1, no asking, no parallel"). **No server-side hard guarantee.**

**Observed host (ChatBox / GPT-5.4) behaviors that `host_instructions` could not prevent:**
- **Parallel batching:** host fires `arrange_day` ×N in one turn, then dumps all day cards at once (forbidden "同回合静默排完 N 天再一次倾倒"). The soft gate (`evaluateArrangePresentGate`) is racy against same-turn parallel calls — all N evaluate `pendingPresent` before any marks → all pass → gate is a no-op against true parallel.
- **Asking confirmation:** host ends its turn after Day N's card and asks "如果你愿意/要不要继续" before calling Day N+1, despite instructions saying "no asking, no waiting for 继续".

**Root cause:** the host's own system prompt / LLM priors override our `host_instructions`. The server cannot control host tool-call ordering, insert presentation between parallel calls, or forbid confirmation questions. The only hard server lever is the soft gate, which (a) loses to the parallel race and (b) cannot block "ask-then-send" since the post-ask call is a legal `dayIndex=last+1`.

**Wording experiments (all failed to fully control host):** "IMMEDIATELY next day" → parallel batching; "in a SEPARATE turn" → host asked "如果你愿意"; "SAME response / never 如果你愿意" → regressed to parallel batching. Settled on step2 "ONE day at a time, no asking" (best observed sequential output, still may ask).

**Implication for future work:** do not over-invest in `host_instructions` wording to enforce host tool-call discipline. If sequential presentation is a hard product requirement, the lever must be elsewhere (e.g., host system prompt config the user controls, or a single-shot `plan_itinerary` path that returns all days at once and lets the host present sequentially from one response). See [ADR-043](../../adr/ADR-043-chatbox-mcp-and-cross-product-closure.md) D9 精简 item 8.

### Testing without ChatBox

Default CI uses HTTP TC-H01–H15 + in-memory MCP parity (TC-H12). See [ADR-019](../../adr/ADR-019-http-first-user-test-automation.md).

## Links

- User test cases: `1.places-agent/agent-specs/8.user-test-cases.md`
- Deployment § MCP: `1.places-agent/agent-specs/6.agent-deployment.md`
- Dual transport: [ADR-003](../../adr/ADR-003-dual-transport.md)
