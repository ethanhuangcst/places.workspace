# ADR-027: Decide reload via SearchCache read API

## Status

Accepted

## Context

what2eat Decide keeps search results in React state (`searchId`, picks, sort). On full page reload, that state is lost even though `SearchCache` rows already exist in PostgreSQL after `POST /api/decide/search`.

List chat transcripts live in browser `localStorage` under `w2e.chat.list.{searchId}`. Without restoring `searchId` and the results list, reload breaks both visible Decide results and chat re-hydration (`history-01` / MVP-3 E2E waits on `[data-testid="decide-results"]` after `page.reload()`).

Alternatives considered:

1. **Re-run search on reload** — wastes vendor quota/latency; may return a different list than the user was viewing.
2. **Client-only persistence (sessionStorage)** — duplicates canonical cache; diverges from server sort/page rules.
3. **Read latest non-expired SearchCache** — reuse existing BFF cache payload; no places-agent call.

## Decision

1. Add **`GET /api/decide/current?page=1`** — authenticated read of the user's latest **non-expired** `SearchCache`, ordered by `createdAt desc`.
2. Response shape matches **`POST /api/decide/search`** (via shared `buildDecideSearchResponse`) plus **`criteria`** (location, meal context, budget, craving, lat/lng) so the Decide form can restore.
3. **404** when no valid cache (`errors.no_search_cache`); client treats as empty Decide.
4. Decide page **hydrates once on mount** when local `result` is still null; URL query params from history rerun **override** cached criteria.
5. Extract **`decide-cache-response.ts`** so reshuffle/sort/current share normalize + slice logic (no third copy of rank/sort/page rules).

Reload does **not** re-fetch places-agent; it only reads DB cache (consistent with ADR-021 live honesty — reload path is not a new vendor probe).

## Rationale

- `SearchCache` is already the source of truth for pagination, sort, and reshuffle; reload should read it, not fork state.
- Restoring `searchId` reconnects list chat keys without storing transcripts server-side (ADR-020).
- Default reload page **1** is sufficient for MVP-3; persisting current page in cache is deferred.

## Consequences

- New contract tests: `tests/api/decide-current.test.ts`.
- `2eat-design.md` §2.3 documents the route.
- E2E `test_mvp3_live.py` reload step passes after hydrate ships.
- Operators must run what2eat with code that includes `/api/decide/current` before signing MVP-3 DoD.

## Date

2026-08-20
