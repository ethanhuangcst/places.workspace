# ADR-058: Cross-provider same-place duplicates — LLM judges at pick time

Family: places-agent stops pool / true-agent loop / dual-route regions (HK).

## Status

**Accepted**（2026-09-07）

## Context

In dual-route regions (Hong Kong: Google + AMAP per ADR-052), the same physical attraction can appear twice — once per provider — with different `native_id` values.

[ADR-056](./ADR-056-registry-backfill-semantics.md) D1 already decided identity is `(provider, native_id)` only: cross-provider rows for the same place stay **separate** in `AttractionPoi`. Read-side [`mergeRegistryPlaces`](../../places-agent/src/core/destination-poi-registry.ts) soft-dedupes by NFKC-normalized name only; when names drift (EN vs ZH, slight vendor wording), the LLM may still see duplicate candidates.

Three options were considered for who owns de-duplication:

1. Read-side clustering (coords + name) before the model sees the pool.
2. Write-side cross-provider merge into one row.
3. **Option A — leave both rows; the loop LLM decides at pick time** (chosen).

## Decision

### D1 — Do not merge on write

Keep ADR-056 D1. Cross-provider same-place stays as two rows. No write-side coordinate or name clustering.

### D2 — Do not hard-dedupe on read

`mergeRegistryPlaces` stays as normalized-name soft dedupe. No new coordinate clustering or second hard gate before the model.

### D3 — De-duplication is a model decision

In the `plan_trip` loop, the LLM judges whether two cards are the same physical place (name / coords / description) and must not schedule both. Code does **not** replace that judgment.

Rationale: true-agent design — the model owns the loop and next hop; code only runs fact gates (eligible, Directions, `(provider, native_id)` uniqueness). Cross-provider “same place?” is a judgment call, not a fact gate.

### D4 — Prompt constraint + existing validation

- Skeleton / fill prompts **must** include a hard rule: do not repeat the same physical place within a day or across days.
- `makeItinerary` validation keeps cross-day uniqueness on `native_id` (M05 / existing). Cross-provider same place has **different** `native_id`s, so that gate does **not** catch them — known residual covered by the prompt rule, not by inventing a fuzzy code gate in this ADR.

### D5 — Known residual and later options

If the model still picks duplicates in practice, a future story may add optional read-side coordinate clustering. That is **out of scope** here and must not silently reintroduce write-side merge.

## Consequences

- Pool row count grows in dual-route regions (expected).
- Slightly more tokens when both vendor cards reach the candidate list.
- Dedup quality depends on prompt + model; aligns with ADR-050 / [`real-agent-refactory.md`](../agent-specs/real-agent-refactory.md) (model holds the loop).
- Operators validating HK probes should watch for same-place double booking under [ADR-057](./ADR-057-cost-conscious-agent-test-strategy.md) L1 cache.

## Alternatives considered

| Alternative | Rejected because |
| --- | --- |
| Read-side coord clustering | Removes candidate from the model’s view; contradicts “LLM judges at pick time” for this gap |
| Write-side cross-provider merge | Breaks `(provider, native_id)` uniqueness; needs fragile coord/name heuristics; conflicts with ADR-056 D1 |

## Related

- [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) — AttractionPoi scope
- [ADR-050](./ADR-050-where2play-no-product-llm.md) — agent-side loop ownership
- [ADR-052](./ADR-052-map-provider-routing.md) — HK dual route
- [ADR-056](./ADR-056-registry-backfill-semantics.md) — identity and upsert
- [ADR-057](./ADR-057-cost-conscious-agent-test-strategy.md) — probe observation of HK duplicates
- [`agent-design.md`](../agent-specs/agent-design.md) §5.2 / §9.2
- [`real-agent-refactory.md`](../agent-specs/real-agent-refactory.md) — ring tools

## Date

2026-09-07
