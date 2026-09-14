# Must-see marking deletion probe (ADR-069)

**Date:** 2026-09-11  
**ADR:** [ADR-069](../../adr/ADR-069-delete-must-see-marking-and-t4-reasons.md)  
**Story:** `agent-discover-110g` (AC Ready)

## Question

Does deleting the must_see marking layer (C) degrade skeleton quality? C injects `[must-see]` tags into skeleton prompt candidate lines, telling the LLM which ≤3 places to prioritize. Without C, the LLM judges importance from pool + constraints + 110e soft preference + persona.

## Method

Probe flag `PROBE_NO_MUST_SEE_TAG=1` patched into `make-itinerary.ts`:
- `candidateLine`: suppress `[must-see]` tag
- `buildFixtureSkeleton`: disable must_see-first sort

Pool composition unchanged (discovery still nominates and grounds the same cards). Only the prompt-side signal is removed. This isolates C's value at the skeleton stage.

5 cases from `specs/agent-specs/prompt-test-case.md`, run twice: with C (baseline) and without C.

## Result

5/5 pass without C. No quality degradation.

| Case | City | Days | Key attraction (C) | Key attraction (no-C) | Verdict |
| --- | --- | --- | --- | --- | --- |
| test1 | 上海亲子 | 3/3 ✓ | Disney full day + 海昌 | Disney full day + 海昌 | Tie |
| test2 | 杭州情侣 | 3/3 ✓ | West Lake + 钱江新城 | West Lake + 龙井 + 西溪 + 太子湾 | no-C slightly better (romantic) |
| test3 | 西安历史 | 3/3 ✓ | 兵马俑 (Day 3) | 兵马俑 (Day 3) + 华清宫 | no-C slightly better (geo cluster) |
| test4 | 里斯本情侣 | 3/3 ✓ | Belém + Oceanário | Belém + Gulbenkian + 植物园 | Tie |
| test5 | 东京动漫 | 3/3 ✓ | Akihabara + Animega(240) | Akihabara (6) + 浅草 | Tie |

## Key observations

1. **兵马俑 3/3 without C** — the critical Xi'an test case passes. The 110e "prioritize well-known attractions" soft preference + "senior itinerary planning expert" persona is sufficient.
2. **Xi'an Day 3 geographic clustering improved without C** — no-C version puts 兵马俑 + 华清宫 together (both in Lintong district, ~30km from city center). With C, 兵马俑 was alone and 华清宫 was not scheduled.
3. **Hangzhou Day 3 more romantic without C** — no-C picks 西溪 + 太子湾 (nature, relaxed) over 钱江新城 (modern city) for couple_romance + relaxed pace.
4. **Theme parks unaffected** — Disney and 海昌 still get full-day dwell (240min) without C. The venue-type dwell tier (110e/ADR-068) handles this, not must_see marking.

## Conclusion

C's `[must-see]` injection is redundant after 110e. The LLM judges importance from the pool + constraints + persona + soft preference without needing code to pre-tag cards. Deleting C removes ~5 modules of hard logic without quality loss.

## Patch reverted

The probe patches were reverted after the test. The actual deletion will be story `agent-discover-110g`.
