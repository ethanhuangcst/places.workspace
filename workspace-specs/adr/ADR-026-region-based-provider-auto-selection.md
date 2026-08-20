# ADR-026: Region-based provider auto-selection

## Status

Accepted — supersedes ADR-005 caller-driven-only routing

## Context

ADR-005 required callers to explicitly pass `providers[]` on every search request. In practice:
- what2eat hardcoded `providersForPin()` which sent both `["AMAP", "GOOGLE_MAPS"]` for mainland China addresses — Google returned US restaurants for Beijing queries
- Callers had to maintain their own region detection logic, duplicating work
- Omitting `providers[]` defaulted to `["GOOGLE_MAPS"]` only, producing wrong results for Chinese addresses

## Decision

1. **places-agent auto-selects providers** based on destination region when caller omits `providers[]`:
   - **大陆** (mainland China) → `AMAP` only
   - **香港** (Hong Kong) → `GOOGLE_MAPS` + `AMAP` + `TRIPADVISOR` enrich
   - **其他** (everywhere else) → `GOOGLE_MAPS` + `TRIPADVISOR` enrich

2. **Caller explicit `providers[]` always overrides** — backward compatible.

3. **Region detection** uses text matching (china-cities list + CJK ratio) and coordinate bounding boxes. Taiwan is explicitly excluded from AMAP (poor coverage). No geocode API call needed.

4. **Callers (what2eat, where2play) should stop passing `providers[]`** and let the agent decide. The `address` field provides enough context for region detection.

## Rationale

The three-region split is simpler than the original locale-based matrix. Locale (EN/CN/HK/TW) does not reliably predict which provider has better data for a given location — a CN user searching Tokyo still needs Google, and an EN user searching Shanghai still needs AMAP.

**Rejected alternatives:**
- Locale-based routing (CN locale → AMAP) — fails for "EN user searching Shanghai"
- Both providers for mainland — Google returns US results for Chinese addresses
- Caller-side detection only — duplicated logic, what2eat bug proved it fragile

## Consequences

- `src/adapters/provider-resolver.ts` and `src/adapters/china-cities.ts` implement the detection
- `src/core/tools.ts` calls `resolveProviderStrategy()` before `fanOut()`
- what2eat `decide-run.ts` no longer passes `providers` — uses `address` for agent detection
- ADR-005's "never geo-force AMAP" is relaxed: agent auto-selects AMAP for mainland, but caller override is still respected

## Date

2026-08-20
