---
title: agent-meal-117 Lisbon / Taipei fill latency probe
type: research
status: active
as_of: 2026-09-20
tags:
  - fill
  - meals
  - google
  - latency
related:
  - ../maps/google-restaurant-search-latency.md
  - ../../agent-specs/agent-test-plan.md
---

# agent-meal-117 — Lisbon / Taipei fill latency

**Purpose:** Wall-clock check for A (search cap) + C (no MCP on restaurant timeout) + B (searchNearby). Not a CI gate.

```bash
cd places-agent
npx tsx --env-file=.env.local scripts/probe-t5-fill-review.ts lisbon taipei
```

## Baseline (before A+C+B) — 2026-09-20

Agent still on searchText × corridor points. `tmp/probe-meal117-baseline.txt`.

| | Taipei | Lisbon |
| --- | --- | --- |
| status / fill | ready 15/15 | ready 18/18 |
| elapsed | 169s | 190.8s |
| fill_s | **102.12** | **118.24** |
| skeleton_s | 29.91 | 21.7 |

Meals were already meal-116 gated (Taipei 4.6–5.0 with review counts).

## After A+C+B — 2026-09-20

Restarted `:3010` with Nearby + early stop. Taipei in `tmp/probe-meal117-after.txt`. Lisbon first after-run failed **skeleton LLM** (day 3 zero attractions, 3 retries) — not Nearby. Retry: `tmp/probe-meal117-lisbon-retry.txt`.

| | Taipei | Lisbon (retry) |
| --- | --- | --- |
| status / fill | ready 17/17 | ready 16/16 |
| elapsed | 138.2s | 136.5s |
| fill_s | **65.88** | **77.72** |
| skeleton_s | 20.85 | 15.18 |

Taipei meal fill samples from agent log: lunch ~5.2s / ~19s / ~5.0s (not minute-class searchText stacks).

### Compare

| Signal | Baseline fill_s | After fill_s |
| --- | --- | --- |
| Taipei | 102s | 66s |
| Lisbon | 118s | 78s |

Skeleton stop count varies with LLM. Fill still includes Directions × modes. Acceptance for A/B/C remains vitest TC-M117-01..07.
