---
title: PlaceCard price_level live probe
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - maps
  - price_level
  - places-agent
  - live-probe
related_spec: 1.places-agent/agent-specs/agent-design.md
related:
  - knowledge/maps/places-capabilities.md
  - knowledge/maps/vendor-adapters.md
---

# PlaceCard `price_level` — live probe

## Summary

places-agent **does return** normalized `price_level` on `POST /v1/search_restaurants` in live vendor mode. AMAP often also returns `price_per_person` (CNY). Coverage is partial — omit when vendor has no price; never invent.

## Evidence (2026-08-20, localhost:3010)

| Probe | providers | cards | with `price_level` | with `price_per_person` |
| --- | --- | --- | --- | --- |
| Clerkenwell, London | `GOOGLE_MAPS` | 20 | **11/20** | 0/20 |
| Central Hong Kong | `GOOGLE_MAPS`+`AMAP` merge | 21 | **6/21** | **5/21** (AMAP) |
| Shanghai | `AMAP` | 20 | **12/20** | **12/20** |

Sample bands observed: `$` … `$$$$` (e.g. Google `$$`/`$$$`; AMAP with cost 33 → `$`, 1283 → `$$$$`).

Contract: field on `PlaceCard` as optional string enum `FREE` | `$` | `$$` | `$$$` | `$$$$`; AMAP may add numeric `price_per_person`.

## Lesson / guidance

- what2eat **decide-09** / **decide-08** By price can rely on agent live data; UI must still show honest missing when the field is absent (~40–70% of cards in these probes).
- what2eat BFF should map `price_level` → `priceLevel` and optionally pass `price_per_person` (PlaceCard/PickDto gap if missing).
- Re-probe after Google fieldMask or AMAP `biz_ext` changes.

## Links

- Design: [`agent-design.md`](../../../1.places-agent/agent-specs/agent-design.md) Price 归一化  
- Capability row 10: [`places-capabilities.md`](./places-capabilities.md)
