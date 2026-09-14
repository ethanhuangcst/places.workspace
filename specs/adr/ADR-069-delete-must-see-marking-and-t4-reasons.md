# ADR-069: Delete must_see marking (C) and T4 must-see reasons (D)

## Status
Accepted

## Context

The must-see subsystem grew into four layers over MVP-7 through MVP-T4 planning:

| Layer | Modules | Role |
| --- | --- | --- |
| **A. Discovery nomination** (110a) | `nominateMustSeeViaLlm`, `buildNominateMustSeeUserMessage` | LLM nominates 20–30 place names → clean → ground → pool. This is the discovery path, not a must-see concept. |
| **B. must_include hard gate** | `must-include-coverage.ts`, `validateSkeleton` | User-typed must-include names from takeoff must appear in skeleton stops. User contract, not agent recommendation. |
| **C. must_see marking** | `find-iconic-places.ts` (dual-mode), `discover-must-see-llm.ts` (`inferMustSeeFromPool`), `markMustSeeByPoolHeat`, `iconic-places-cache`, `must_see` boolean on PlaceCard / trip-store, `[must-see]` prompt injection | Code marks ≤3 cards `must_see=true` → skeleton prompt shows `[must-see]` → LLM prioritizes. ~5 modules of hard logic. |
| **D. T4 user-facing reasons** (planned, not built) | `agent-itinerary-101`, `2play-plan-102` | Show must-see reasons to user; chat refine skeleton. Whole MVP slice. |

C violates the true-agent principle (agent-builder: "trust the model, get out of the way"). Code pre-decides which 3 places are "must-see" and tells the LLM to prioritize them — pre-specifying workflow in a domain the LLM can already judge. ADR-043's own retrospective already flagged `ensureHardMustSeeCoverage` / deterministic injection as over-engineered.

After 110e (ADR-068), the skeleton prompt already carries a destination-agnostic soft preference "prioritize well-known attractions matching trip constraints" plus a "senior itinerary planning expert" persona. Probe evidence (below) shows this is sufficient without C's `[must-see]` injection.

D (T4 must-see reasons + chat refine) is a parallel interaction channel that duplicates what T8 chat refine already plans to do — let the user ask "why this not that" in natural language and adjust the itinerary.

## Decision

**Delete C and D. Keep A and B.**

### Delete

| Item | Disposition |
| --- | --- |
| `find-iconic-places.ts` + tests | Delete |
| `discover-must-see-llm.ts` (`inferMustSeeFromPool`) + tests | Delete |
| `pool-heat-must-see.ts` (`markMustSeeByPoolHeat`, `applyNominatedMustSee`) + tests | Delete |
| `iconic-places-cache.ts` + tests | Delete |
| `must_see` boolean on `PlaceCard` | Remove field |
| `must_see` merge/persist logic in `trip-store.ts` | Remove |
| `[must-see]` tag injection in `buildSkeletonUserMessage` (`candidateLine`) | Remove |
| `must_see`-first sort in `buildFixtureSkeleton` | Remove |
| `inferred_must_see` return from `discoverPlaces` | Remove |
| `travel_tips` iconic branch (`findIconicPlaces` call) | Replace: use skeleton stops (already scheduled = already judged important) |
| T4 `agent-itinerary-101` / `2play-plan-102` | Remove from backlog; chat refine moves to T8 |
| Feature 49 (`agent-iconic-49`) | Superseded by ADR-069 |
| Feature 74 (`agent-iconic-74`) | Superseded by ADR-069 |

### Keep

| Item | Reason |
| --- | --- |
| `nominateMustSeeViaLlm` / `buildNominateMustSeeUserMessage` (A) | Discovery path (110a). Nominates ALL places, not just must-see. Name contains "must-see" but function is discovery. |
| `must_include` field + `must-include-coverage.ts` + `validateSkeleton` hard gate (B) | User-typed contract, not agent recommendation. |
| `buildConstraintGlossary` (110e) | Constraint explanation, not must-see marking. |
| 110e "prioritize well-known attractions" soft preference | Replaces C's `[must-see]` injection. |
| 110e "senior itinerary planning expert" persona | Replaces C's quality signal. |

### travel_tips migration

`travel_tips` currently calls `findIconicPlaces` for its `iconic_places` output. After deletion:
- `iconic_places` source changes to skeleton stops (top 3 attraction names by day order).
- `iconic_grounded` becomes always `true` (skeleton stops are grounded by definition).
- No LLM call for iconic inference; tips-prose LLM call remains.

## Probe evidence

5 cases from `specs/agent-specs/prompt-test-case.md`, run with `PROBE_NO_MUST_SEE_TAG=1` (C's `[must-see]` injection disabled, pool unchanged):

| Case | City | Days (C / no-C) | Key attraction (C) | Key attraction (no-C) | Verdict |
| --- | --- | --- | --- | --- | --- |
| test1 | 上海亲子 | 3/3 ✓ | Disney full day + 海昌 full day | Disney full day + 海昌 full day | Tie |
| test2 | 杭州情侣 | 3/3 ✓ | West Lake + 灵隐 + 钱江新城 | West Lake + 灵隐 + 龙井 + 西溪 + 太子湾 | no-C slightly better (more romantic) |
| test3 | 西安历史 | 3/3 ✓ | 兵马俑 ✓ (Day 3) | 兵马俑 ✓ (Day 3) + 华清宫 | no-C slightly better (geographic clustering) |
| test4 | 里斯本情侣 | 3/3 ✓ | Belém day + Oceanário | Belém day + Gulbenkian + 植物园 | Tie |
| test5 | 东京动漫 | 3/3 ✓ | Akihabara + Animega(240) | Akihabara (6 stops) + 浅草 | Tie |

**Result:** 5/5 pass without C. No quality degradation. Some cases slightly better without C (Xi'an Day 3 geographic clustering, Hangzhou Day 3 romantic tone). The 110e soft preference + persona is sufficient.

## Rationale

1. **True-agent principle:** C is code pre-deciding importance for the LLM. The LLM can judge from the pool + constraints + persona. ADR-068 already established "trust the model for density and implicit must-sees."
2. **Redundancy:** After 110a, discovery nominates 20–30 grounded places. After 110e, the prompt carries "prioritize well-known attractions" + persona. C's `[must-see]` tag is a second judgment layer on top of what the LLM already does.
3. **Maintenance cost:** ~5 modules + tests for a signal that has a lighter replacement.
4. **ADR-042 alignment:** `findIconicPlaces` ungrounded mode has LLM produce place names from parametric knowledge — compliant but close to the city-encyclopedia boundary. Deleting it is cleaner.
5. **T4 replaced by T8:** "Why this not that" is a natural chat question. A dedicated must-see-reasons interaction is a parallel channel that T8 chat refine already covers.
6. **must_include unaffected:** B (user-typed contract) is independent of C. Deleting C does not affect the user's "I typed these, you must schedule them" guarantee.

## Consequences

- Skeleton prompt no longer shows `[must-see]` tags; LLM judges importance from pool + constraints + persona + 110e soft preference.
- `travel_tips` `iconic_places` comes from skeleton stops, not a separate LLM inference.
- 2play removes `must_see` badge from cards (if present).
- T4 (`agent-itinerary-101` / `2play-plan-102`) removed from backlog; chat refine consolidated into T8 (`agent-chat-93e` / `2play-plan-90e`).
- Feature 49 / 74 marked Superseded.
- `agent-discover-110f` (implicit must-see monitor) remains — it monitors the 110e soft preference, not C.
- `must_include` hard gate in `validateSkeleton` unchanged.

## Supersedes

- Feature 49 (`agent-iconic-49` findIconicPlaces dual-mode) → Superseded
- Feature 74 (`agent-iconic-74` findIconicPlaces must-see quality) → Superseded
- ADR-062 T4 slice (must-see reasons + chat refine) → T4 removed; chat refine moves to T8
- ADR-065 B clause (nominate produces `must_see` + reasons for T4 display) → nominate (A) stays for discovery; `must_see` marking and reasons display removed

## Related

- [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) — no city POI encyclopedia
- [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md) — LLM-driven discovery (A)
- [ADR-068](./ADR-068-soft-rhythm-vs-hard-safety-rails.md) — soft rhythm vs hard safety rails (110e)
- [ADR-062](./ADR-062-mvp-t3-skeleton-vs-t4-nominate.md) — T3 vs T4 split (T4 removed by this ADR)
- [ADR-065](./ADR-065-nominate-vs-skeleton-relationship.md) — nominate vs skeleton (B clause superseded)

## Amendment 2026-09-14 — Product policy reconfirmed

**Decision:** Do **not** build any further **agent prompt must-see** product features.

| Keep | Do not build / do not revive |
| --- | --- |
| A — LLM discovery nomination (110a), no `must_see` flag | C — must_see marking / `[must-see]` injection |
| B — user `must_include` hard gate | D — T4 must-see reasons UI / dedicated must-see chat |
| ADR-068 soft preference "prefer well-known attractions" as **planning judgment** (not labeled must-see) | New stories whose product concept is "agent 提示必去点" |

Rationale: C/D already deleted; reopening them reintroduces code/UI that pre-decides importance. Chat refine for "why this place" belongs in T8 only.

## Date
2026-09-11 · amended 2026-09-14
