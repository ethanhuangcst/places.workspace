---
title: Scenic amenity children must not enter attraction pool
type: ops-lesson
status: active
as_of: 2026-09-21
tags:
  - discover
  - eligible-attraction
  - amap
  - skeleton
related_spec: specs/adr/ADR-042-no-city-encyclopedia-in-source.md
related:
  - adr/ADR-038-discover-places-quality.md
  - knowledge/agent/plan-return-hydrate-artifacts.md
---

# Scenic amenity children must not enter attraction pool

## Summary

AMAP often returns scenic **facility** POIs (母婴室、卫生间、寄存处) whose names contain a real landmark prefix. Substring grounding (`pickNominatedGroundCard`) can treat them as hits for nominate queries like「西溪湿地」. They must be denied as service fragments before they enter `candidates.places`, or the skeleton LLM will schedule them as attractions (pool-only).

## Evidence

- Hangzhou couple_romance skeleton included `西溪湿地洪园景区母婴室` without kids trip type.
- Cause: fragment deny lacked nursing/toilet tokens; eligibility is intentionally looser than `ATTRACTION_ALLOW` so `*景区` titles survive.
- Not caused by a recent skeleton-overlay regression; Visa/geocode changes did not touch this path.

## Lesson / guidance

1. Keep amenity vocabulary in `ATTRACTION_FRAGMENT_DENY` (destination-agnostic templates), not city POI lists.
2. Nominate ground already skips `isAttractionServiceFragment` — extending that regex covers both pool gate and grounding.
3. Prefer parent scenic cards when both amenity and landmark substring-match; never invent Hangzhou rows.

## Links

- Code: `places-agent/src/core/place-filters.ts`, `eligible-attraction.ts`
- Plan RCA: Hangzhou skeleton RCA (couple_romance)
