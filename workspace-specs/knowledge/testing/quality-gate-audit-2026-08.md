---
title: Quality gate audit — fixture gaps and ghost tests
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - testing
  - quality-gates
  - fixture
  - provider-resolver
related_spec: 1.places-agent/agent-specs/agent-test-plan.md
related:
  - adr/ADR-021-live-vendor-no-fixture.md
  - adr/ADR-026-region-based-provider-auto-selection.md
  - knowledge/testing/vendor-live-vs-fixture.md
---

# Quality gate audit — fixture gaps and ghost tests

## Summary

Post-MVP-2 audit of places-agent quality gates found five categories of gap where test coverage on paper (spec docs) did not match reality (runnable code). All were fixed in the same session. The core lesson: **a test case in a spec doc with no corresponding Makefile target or test file is worse than no spec** — it creates false confidence.

## Evidence

### 1. Ghost E2E tests (TC-E2E-01~08)

`agent-test-plan.md` §19 defined 8 caller simulation E2E tests and referenced `make test-e2e-caller`. Neither the Makefile target nor any test file existed. The spec created the impression of end-to-end caller verification that was never running.

**Fix:** Implemented `scripts/test-e2e-caller.sh` (bash + Python assertions, matching `verify-*.sh` pattern) + Makefile target. Opt-in, not in `make quality`.

### 2. Fixture-coupled assertions (TC-H04 / TC-H14)

TC-H04 asserted exactly "3 cards: Yat Lok, Tim Ho Wan, 太興燒味". TC-H14 asserted "2 cards: Yat Lok, Tim Ho Wan". These tested "fixture returned fixture data", not merge/filter behavior. If the fixture file changed, these tests would fail — but they would also pass with completely broken merge logic as long as the fixture stayed the same.

**Fix:** Replaced with behavioral assertions: merge reduces count, both providers appear in sources, all cards have required fields. No specific names or counts.

### 3. Fixture geocode default-to-HK trap

`resolveFixtureGeocode()` had branches for Shanghai and Tokyo, then a catch-all defaulting to Hong Kong. "北京三里屯" in fixture mode geocoded to HK (22.28°N) — correct for HK, wrong for Beijing. No test caught this because the fixture search also returned HK restaurants for HK coordinates.

**Fix:** Added 6 more city branches (北京, 成都, 广州, 澳门, Lisboa, Singapore) + 9 unit tests.

### 4. CJK heuristic misdetecting overseas cities

`provider-resolver.ts` used CJK character ratio > 30% as a mainland China heuristic. "新加坡滨海湾" (100% CJK) was routed to AMAP — an API that has no Singapore data.

**Fix:** Added `OVERSEAS_CJK_MARKERS` exclusion list (18 overseas Chinese city names) checked before the CJK ratio heuristic. Added 9 boundary tests including Macau, Singapore, coordinate edges.

### 5. Tripadvisor multi-result matching untested

`pickLocation()` correctly handles multiple nearby results (picks best name-match score, tie-breaks by distance), but no test exercised this path — the fixture always returned exactly one matching result.

**Fix:** Added 2 test cases: multi-result with competing partial matches, and all-different-names (no match).

## Lesson / guidance

1. **Spec-first quality gates need a mechanical check**: if a spec references `make <target>`, verify the target exists before marking the spec "active". A CI job that runs `make -n <target>` on all Makefile references in spec docs would catch this.

2. **Fixture-mode tests should assert behavior, not fixture content**: "expect exactly N cards with these names" is a fixture regression test, not a behavioral test. Good fixture tests assert structural properties (merge reduces count, provider field matches request, coordinates in range).

3. **Fixture geocode catch-all defaults are dangerous**: a default that silently returns valid-looking coordinates for any input masks real geocode failures. Each UAT city in the test plan should have a fixture branch, or the default should return an obviously wrong sentinel (e.g., lat 0, lng 0) that fails downstream.

4. **Text heuristics need negative tests for the same script family**: CJK ratio works for mainland Chinese addresses but catches Japanese, Korean, Thai-Chinese, and Singapore-Chinese text. Any heuristic based on script detection needs explicit negative cases for the most common false positives.

5. **The opt-in live gate pattern (`verify-*.sh`) works — extend it, don't replace it**: the existing bash+Python assertion scripts are simple, debuggable, and match the Makefile convention. `test-e2e-caller.sh` follows the same pattern.

## Links

- [ADR-021 — live vendor no fixture](../../adr/ADR-021-live-vendor-no-fixture.md)
- [ADR-026 — region-based provider auto-selection](../../adr/ADR-026-region-based-provider-auto-selection.md)
- [vendor-live-vs-fixture.md](./vendor-live-vs-fixture.md) — earlier lesson about fixture honesty
