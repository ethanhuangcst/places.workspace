---
title: Skeleton eligibility — transit/auto noise vs 娱乐场所
type: ops-lesson
status: active
as_of: 2026-09-10
tags:
  - eligible-attraction
  - place-filters
  - adr-066
  - agent-itinerary-108
related:
  - adr/ADR-066-venue-type-allowlist-vs-city-poi.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - knowledge/agent/eligible-attraction-before-registry.md
---

# Skeleton eligibility — transit/auto noise vs 娱乐场所

## Summary

AMAP often returns satellite POIs whose **names** contain venue words (主题公园、博物馆) but whose **categories** are parking / EV charging / bus stop. `isEligibleAttraction` used to admit them because it only ran fragment + dining checks, not a category noise gate.

## Evidence (Shanghai kids verify, 2026-09-10)

- `张江主题公园停车点` — category `交通设施服务;停车场` (name uses 停车点, not 停车场)
- `小鹏超级充电站(…张江主题公园站)` — category `汽车服务;充电站`
- `上海博物馆站(公交站)` — category `交通设施服务;公交车站`

After `agent-itinerary-108`: fragment deny adds 停车点/充电站/公交站; `isEligibilityNoiseCategory` rejects `交通设施` / `汽车服务` only.

## Lesson / guidance

- **Do not** wire `isNoiseCategory` into `isEligibleAttraction`. It denies `娱乐场所`, which would undo 105 (迪士尼乐园 / 欢乐谷 under entertainment).
- Use a **narrow** category gate (`isEligibilityNoiseCategory`) for skeleton eligibility.
- Keep fragment tokens destination-agnostic (ADR-042/066); extend when vendor naming drifts (停车点 ≠ 停车场).
- Prompt ranking (107) already schedules park-shaped cards when they are in the pool; eligibility leaks hurt signal-to-noise more than prompt wording.

## Out of scope (follow-ups)

- Pool bloat from registry merge (~300 cards)
- Vendor search not returning major theme parks for `主题公园`
