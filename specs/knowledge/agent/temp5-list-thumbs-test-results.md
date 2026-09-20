# Temp 5 — list thumbs from fill `photos[0]` (test runs)

Destination-agnostic class: timeline reads slim pool + fill `stop_display.card.photos[0]`; place-sheet uses live Details (ADR-051 D1–D2).

## Pre-fix (expected / code review)

| Case | `slimCandidatesForStore` | Fill `photos[0]` |
| --- | --- | --- |
| AMAP `http://store.is.autonavi.com/...` only | **Dropped** (`startsWith("https://")` gate) | Empty unless Details resolve |
| Google media stub + lh3 https | Keeps lh3 | Unchanged |
| `cdn.example.com` placeholder | Kept https (old slim did not reject placeholders) | N/A for this story |

Repro: Hangzhou / 龙井村 — pool lost http photo at slim → fill copied empty `photos[]`; sheet still showed image via `get_place_details`.

## Post-fix run (2026-09-20)

Command:

```bash
cd places-agent && npm test -- --run \
  src/core/trip-store.test.ts \
  src/core/plan-next-stop-stay-photo.test.ts \
  src/core/resolve-display-photo.test.ts
```

| Metric | Result |
| --- | --- |
| Test files | 3 passed |
| Tests | 33 passed |
| Duration | ~3.7s |

New cases:

- `trip-store.test.ts`: AMAP http→https in slim; drop `cdn.example.com` in slim
- `plan-next-stop-stay-photo.test.ts`: Temp 5 pool https copy by native_id; Details writes photo when pool has none

## Comparison

| Case | Pre-fix | Post-fix |
| --- | --- | --- |
| AMAP http in trip candidates slim | No `photos[]` | `https://store.is.autonavi.com/...` |
| Placeholder host in slim | Could persist | Skipped; next displayable kept |
| Fill pointer + pool https | OK | OK (explicit Temp 5 test) |
| Fill pointer + no pool photo | Details path (existing) | Asserted same-provider Details |

**Out of scope verified unchanged:** 2play list does not call Details; old SavedItinerary / draft without replan still has empty thumbs (ADR-051 D5).

## Verify re-run (2026-09-20)

```bash
cd places-agent && npm test -- --run \
  src/core/trip-store.test.ts \
  src/core/plan-next-stop-stay-photo.test.ts
```

| Metric | Result |
| --- | --- |
| Test files | 2 passed |
| Tests | 30 passed |
| Duration | ~3.4s |

Matches post-fix run; issue class resolved for new plans after agent deploy + replan.

## Live Hangzhou check (2026-09-20) — 平湖秋月

Temp 5 slim (`http→https`) **did not** cover this POI.

| Fact | Evidence |
| --- | --- |
| Trip candidates `平湖秋月` | `native_id` `B023B024F8`, **no `photos[]`** |
| AMAP Place Detail | `errors.place_not_found` (tip / inputtips id, `pois:[]`) |
| Sheet photo | 2play `resolvePlaceDetailsWithNameFallback` → `search_places` 「平湖秋月」→ e.g. 碑亭 `B0HU44IKRD` https |
| List | fill never wrote `photos[0]` (Details empty; no name-search on agent) |

**Follow-up:** fill copies a name-overlapping search photo onto the pool card **without** changing `native_id` (ADR-072). New plan required.
