# ADR-056: AttractionPoi registry backfill semantics

Family: places-agent stops pool / `plan_trip` commit.

## Status

**Accepted**（2026-09-07）

## Context

City stops pool (`Destination` + `AttractionPoi`) is the durable, destination-agnostic cache of verified attractions (ADR-049). The as-built `discover_places` path called `safeUpsertEligiblePois`; the true-agent `plan_trip` commit path did not, so Lisbon/Hangzhou probes left the registry empty.

Three open questions needed product decisions:

1. How to match a map search hit to an existing pool row.
2. What to do on match with / without field diffs, and on no match.
3. Whether per-trip `must_see` belongs in `cardSlim`.

## Decision

### D1 — Identity is `(provider, native_id)` only

Within a destination, uniqueness is Prisma `@@unique([destinationId, provider, nativeId])`.

- Match key comes from `registrableNative(card)` — first source with a non-empty `native_id`.
- No name or coordinate fuzzy match. Names and coords drift; vendor place ids are stable.
- Cards without `native_id` are not registered (trip candidates only).
- Cross-provider rows for the same physical place stay separate (ADR-049 D9 / same-identity stack).

### D2 — Three update cases

| Case | Condition | Action |
| --- | --- | --- |
| Match + no diff | Same `(provider, native_id)`; name / lat / lng / `photos[0]` / rating equal | Skip DB write |
| Match + diff | Same key; any compared field changed | Update row; on name change push old name into `aliases`; keep `details` / `detailsFetchedAt` |
| No match | Key not in pool | Insert row |

Diff compare uses NFKC-normalized name, coordinate epsilon `1e-6`, and first displayable https photo (same filter as trip slim).

### D3 — `cardSlim` stores facts, not per-trip flags

- **Store:** provider, name, location, sources (with `native_id`), rating, `photos[0]` (resolved https only — ADR-051).
- **Do not store:** `must_see`. That flag is a per-trip heat / chip decision. Persisting it leaks User A's selection into User B's skeleton ranking because `makeItinerary` merges registry cards and does not re-run `findIconicPlaces` / reset heat.

### D4 — Wire point

`plan_trip` `commitTripInternal`: after `resolveDisplayPhotosForCards`, before `dualWriteTrip`, call `safeUpsertEligiblePois(withPhotos, { city, lat, lng })`. Failures stay swallowed (`safeUpsert*`) so registry miss never fails the trip.

## Consequences

- True-agent intake fills the city pool; later trips can merge registry places into the skeleton pool without re-hitting media for known `photos[0]`.
- Diff-skip reduces no-op writes on probe re-runs / repeated commits.
- Callers must treat `must_see` as trip-local; heat-rank or chip selection sets it each run.
- Orphan rows (vendor rotates `native_id`) are left in place; no automatic delete in this ADR.

## Related

- [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) — eligible attractions, registry scope
- [ADR-051](./ADR-051-discover-resolve-display-photo.md) — display photo resolve + store
- [agent-design.md](../agent-specs/agent-design.md) — 真智能体 commit hook + 落库（历史稿指针：`real-agent-refactory.md` stub）
