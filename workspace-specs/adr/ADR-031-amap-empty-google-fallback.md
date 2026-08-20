# ADR-031: Empty AMAP → one Google search fallback

## Status

Accepted

## Context

ADR-026 auto-selects **AMAP only** for mainland destinations when callers omit `providers[]`. Chat (after stripping LLM-supplied `providers`) and Decide pins often force this path via `near` / Shanghai context.

Live incident (2026-08-20): user query typo **吴记鲜定位** (real venue **吴记鲜定味**). With Shanghai `near`, AMAP returned empty; Google fuzzy-matched the correct shop. Before chat stripped `providers`, the model often forced Google and “found” the place — after ADR-026 + chat strip, the same typo looked like a regression.

Caller-explicit `providers: ["AMAP"]` must remain absolute (debug / forced vendor tests).

## Decision

1. When `providers[]` were **auto-selected** (caller omitted them) and the resolved set is **AMAP-only**, and the first search returns **zero cards**, places-agent runs **one** additional search with `GOOGLE_MAPS`.
2. Explicit `providers[]` from the caller **never** triggers this fallback.
3. Applies to `search_restaurants` and `search_places` (`shouldTryGoogleAfterEmptyAmap` in `src/core/tools.ts`).

## Rationale

- Prefer AMAP on mainland (ADR-026) when it has hits.
- Preserve discoverability for typos / POI names AMAP misses that Google fuzzy-matches.
- Avoid undoing ADR-026 for the common path (Google US junk when both vendors run in parallel on Chinese addresses).

**Rejected:** Always dual-search mainland; fall back on any AMAP error (not only empty); chat-only fallback (Decide benefits too).

## Consequences

- Slight extra latency / quota on empty AMAP auto searches.
- Chat typo cases like 定位→定味 recover without forcing Google as primary.
- Docs: [`knowledge/web-app-development/what2eat-chat-provider-auto-select.md`](../knowledge/web-app-development/what2eat-chat-provider-auto-select.md).

## Date

2026-08-20
