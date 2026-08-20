# what2eat MVP-3 — lessons

Captured during MVP-3 (list chat, place chat, History) implementation and DoD close on 2026-08-20.

## What shipped

- `POST /api/chat` BFF proxy to places-agent `/v1/chat` (list + place scopes); no DB transcript rows.
- Browser `localStorage` keys `w2e.chat.list.{searchId}` and `w2e.chat.place.{provider}:{nativeId}`; cleared on logout.
- Decide: floating `AgentChatPanel` (`agent-chat-open`, `agent-chat-close`, `?open=chat`).
- Place details: `.place-why-chat` with `place-chat-input` / `place-chat-send`.
- History: `GET/POST /api/history` (outcome always `went`); `/history` page; link from Saved toolbar.
- **`GET /api/decide/current`** — reload hydrate from non-expired `SearchCache` + criteria ([ADR-027](../../adr/ADR-027-decide-searchcache-hydrate.md)).
- **`decide-cache-response.ts`** — shared normalize/slice for search, reshuffle, sort, current.
- **Save → history** on server ([ADR-028](../../adr/ADR-028-decision-history-on-save.md)); Decide `place-save` save-only.

History Skipped removed from product specs (went-only).

## E2E failure modes (live probe debugging)

| Symptom | Root cause | Fix |
| --- | --- | --- |
| `decide-results` timeout after `reload()` | React `result` lost; no read API for `SearchCache` | `GET /api/decide/current` + mount hydrate |
| `chat-agent-msg` timeout after place chat | Closed list chat still rendered composer bubbles in DOM | Render composer only when panel `open` |
| `history-row` timeout | Client `recordWent` race + repeat run unsave path | Server history on `POST /api/saved`; save-only button |

**Harness note:** If `:3020` serves stale code, E2E may pass hydrate unit tests but fail live. Stop old dev server or let `with_server.py` start a fresh `npm run dev` before `make test-e2e-mvp3-live`.

## Live probe

Run `make test-e2e-mvp3-live` with places-agent on `:3010`, valid caller key, and `PLACES_AGENT_TIMEOUT_MS=90000`. Chat replies require agent `OPENAI_*` keys; E2E asserts structure, not fixed copy.

Journey: login → Decide search → list chat → **reload** (results + chat keys) → place chat → save → History → rerun Decide → logout clears chat keys.

**Recent live pass:** 2026-08-20, Clerkenwell, London (`mvp3.live@what2eat.food`).

**DoD status:** **Complete** — user confirmed usable; `make test-e2e-mvp3-live` green.

## Operator checklist

1. Postgres `:5435` up (`make up` in what2eat).
2. places-agent + what2eat dev servers on `:3010` / `:3020` (or rely on `e2e/run.py` to start missing services).
3. `make test` (122+ cases) + `make test-e2e-mvp3-live` before DoD sign-off.

## ADRs

- [ADR-027](../../adr/ADR-027-decide-searchcache-hydrate.md) — reload hydrate via SearchCache read API.
- [ADR-028](../../adr/ADR-028-decision-history-on-save.md) — decision history on save; save-only Decide control.

Follows ADR-020 (HTTP-only chat, browser-local transcripts) and ADR-023 (Postgres history table).
