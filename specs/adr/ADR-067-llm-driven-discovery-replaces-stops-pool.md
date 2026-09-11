# ADR-067: LLM-driven discovery replaces code-template stops-pool

**Status:** Accepted  
**Date:** 2026-09-11  
**Supersedes:** ADR-062 (T3/T4 切分), ADR-065 (nominate vs skeleton relationship)  
**Decided by:** Ethan Huang, after 6-item diagnostic + 2 probe runs (5-group A/B/OptA + NoEnum test1)

## Context

MVP-T3 skeleton path uses code-template search (`skeletonPoolQueries` → `searchPlaces` → pool → LLM picks from pool). Six quality issues were diagnosed:

1. **找景点路径设计** — code templates can't consider season/trip-type when searching; 断桥残雪 (winter sight) gets into a September pool.
2. **季节提示词两套规则冲突** — nominate hard rule vs skeleton soft rule ("don't hard-drop for season").
3. **骨架提示词选哪套** — traveler block + season rule design unclear.
4. **11 个起飞输入潜台词** — only family_kids gets venue-type hint; `other` labeled as "preference" only.
5. **3 天漂移到 5 天** — `ensureFarClustersOwnDays` peels far clusters onto new days without checking numDays.
6. **北大壶挂起** — small ski town, AMAP/Google POI coverage too thin to fill 3-day skeleton.

## Decision

### Core: LLM-driven discovery (Option A)

Replace code-template stops-pool discovery with LLM-driven discovery:

```
geocode(city)
  → LLM nominates 20-30 place names (JSON array, from trip context)
  → clean (drop non-attraction types + drop un-groundable vague names)
  → searchPlaces each name → grounded POI cards (write trip.candidates + registry cache)
  → LLM schedules skeleton from grounded cards
  → code validates (no repair) → commit
```

**Delete:** `skeletonPoolQueries`, `expandPlacesForSkeleton`, `intakeEligible` template-search path.  
**Keep:** registry as POI card cache (grounding checks cache before searchPlaces); `enrich` registry read stays.  
**Cost:** 2x LLM latency/cost; hallucination fallback (names that don't ground are dropped; over-nominate to compensate).

### todo1 — Option A (this ADR)

T3 uses the nominate capability for internal discovery. T4 still uses the same capability for user-facing must-see. New ADR supersedes ADR-062 (T3/T4 split) and ADR-065 (nominate independent).

### todo2 — Season rule: 2a-none

Skeleton prompt **deletes the season rule section entirely**. Season is a discovery concern (nominate LLM applies hard rule at source — "don't list seasonal-only sights not visible then"). The scheduling LLM sees the travel month in tripLine and can self-judge; the scheduling prompt does not repeat the discovery's job.

### todo3 — Discovery prompt style: OptA (single message + 3 patches)

Probe tested 3 arms across 5 test cases (`scripts/probe-discover-style-ab.ts`, results `tmp/probe-discover-style-ab.json`):

| Arm | Style | User-quality result |
| --- | --- | --- |
| A | Single user message, no system prompt (current nominate) | Wins 3/5 on landmark capture |
| OptA | A + 3 patches: 数量上限 20-30 + 主题优先 + 反模糊加强 | Wins 5/5 |
| B | System prompt (base + discovery overlay) + user message | Wins 2/5, misses icons (Disneyland, Regaleira) |

**Adopt OptA.** Three patches to `buildNominateMustSeeUserMessage`:
1. **数量上限**: "推荐约 20 至 30 个地点" — bounds output, keeps pool thick for grounding.
2. **主题优先**: "优先提名符合行程类型和同行人偏好的地点，再补该城市通用经典景点" — see todo4 a3: no enumeration.
3. **反模糊加强**: "不要历史文化街区、区域、商圈名，只写具体可搜索的地点" — note: partially ineffective under 20-30 count pressure (LLM fills with 西湖十景/古镇); vague names are dropped at grounding, acceptable.

### todo4 — Takeoff 11 fields subtext: a3 + b2

- **a3 trip_type 全去枚举**: No venue-type enumeration for any trip_type (including family_kids). Probe NoEnum verified test1 上海亲子: without "乐园/动物园/水族馆" enumeration, LLM still nominated all core kid spots (Disneyland/海昌/野生动物园/动物园/水族馆/科技馆/天文馆/自然博物馆 = 8, enough for 3 days). Enumeration's extra 5 spots are redundant for a 3-day trip. Trust the model to infer venue type from the trip_type label.
- **b2 other 去"偏好"二字**: `other` label changes from "其他（偏好）" to "其他", removing the preference-only implication. LLM can read negations ("不去博物馆"); no explicit prefer/avoid distinction added.

### todo5 — Far-cluster drift: validate, don't repair + transparent display

Abandon silent code repair (`ensureFarClustersOwnDays` etc.). New model:
1. LLM respects all user conditions (numDays, don't mix far districts same day). When it can't, it attaches `deviations` (non-conformance + reason) to the return, written to DB.
2. Code validates the LLM output against boundary conditions, lists non-conforming facts only, does not judge reasons, does not repair.
3. Non-conformances + LLM reasons shown as text below the skeleton in the existing Assistant window (no warning panel). User evaluates.

Skeleton JSON contract adds optional `deviations` field (Zod schema). Code reports facts, LLM provides reasons, user judges.

### todo6 — 北大壶 hang: 6a transparent deviation + 6b grounding radius (user confirm)

Root cause: 北大壶 is a small ski town; AMAP/Google POI coverage too thin (probe found 3 usable attractions, 1 closed). 3-day skeleton can't fill → validation fails → retry → timeout (hang). Not a geocode failure.

- **6a**: Use todo5 transparent deviation — when POI insufficient, LLM returns deviation "仅 N 个景点，无法填满 numDays", Assistant window text explains, user evaluates.
- **6b**: Grounding layer, when local POI insufficient, auto-expand search radius to nearby (e.g., 50km) — but **requires user confirmation** before including nearby-city attractions (no auto-mixing).

## Consequences

| Area | Change |
| --- | --- |
| Discovery path | `skeletonPoolQueries` / `expandPlacesForSkeleton` / `intakeEligible` deleted; LLM nominate + grounding replaces |
| Skeleton prompt | Season rule section deleted (2a-none); `other` label drops "偏好" (b2) |
| Nominate prompt | 3 patches added (数量 20-30 + 主题优先无枚举 + 反模糊) |
| Code repair functions | `ensureFarClustersOwnDays` etc. change from repair to validate-only; no silent mutation of LLM output |
| Skeleton JSON | Add optional `deviations` field |
| DB | Store deviation reasons |
| UI | Assistant window shows deviations as text below skeleton (no new panel) |
| Grounding | POI-insufficient → expand radius with user confirmation |
| ADR-062 | Superseded — T3 now uses nominate capability for internal discovery |
| ADR-065 | Superseded — nominate capability shared between T3 (internal) and T4 (user-facing) |
| Cost | 2x LLM latency/cost for discovery + scheduling |

## Compliance

- **ADR-042 (no city encyclopedia)**: LLM nomination is not a city→POI table; venue-type hints removed (a3). Compliant.
- **ADR-050 (where2play no product LLM)**: LLM runs in places-agent, not BFF. Compliant.
- **ADR-066 (venue-type allowlist)**: a3 removes enumeration entirely; no venue-type allowlist in prompt. Compliant.
- **True-agent principle**: Code provides capabilities (geocode, searchPlaces grounding, validation); model provides judgment (discovery, scheduling, deviation reasons). Compliant.

## Implementation

As `agent-discover-110` (next batch, after current T3+ quality stories close). Not implemented now — current T3+ still runs code-template path. This ADR records the decided direction.
