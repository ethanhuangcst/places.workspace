---
title: Probe file cache must not strip CJK from filenames
type: ops-note
status: active
as_of: 2026-09-07
tags:
  - testing
  - amap
  - cache
related:
  - adr/ADR-057-cost-conscious-agent-test-strategy.md
---

# Probe file cache CJK filename collision

`PLACES_PROBE_CACHE_DIR` filenames used to replace every non-ASCII character with `_`.  
`geo|杭州||` and `geo|西安||` both became `geo______.json`, so later city seeds reused Hangzhou coordinates.

**Rule:** file keys must include a hash of the full cache key (and may keep a CJK-readable prefix). In-memory keys were already unique; only the file layer collided.

**Symptom:** `seed-city-pois.ts` prints the same `anchor:` lat/lng for 杭州 / 西安 / 上海.
