---
title: agent-meal-116 Hangzhou / Taipei meal-rank probe
type: research
status: active
as_of: 2026-09-20
tags:
  - fill
  - meals
  - probe
  - agent-meal-116
related:
  - ./research_fill_rule_meals.md
  - ../../agent-specs/agent-test-plan.md
---

# agent-meal-116 — Hangzhou / Taipei fill meal probe

**Purpose:** Before/after live check that fill meal pick prefers rating gates over nearest low-score canteen. Not a CI gate; AC uses fixture cards in `meal-corridor.test.ts`.

## Cases (`probe-t5-fill-review.ts`)

| id | City | Locale | Days | Origin | Provider (ADR-052) |
| --- | --- | --- | --- | --- | --- |
| `hangzhou` | 杭州 | CN | 3 | SFEEL设计师酒店(杭州西湖武林广场店) | AMAP |
| `taipei` | 台北 | TW | 3 | 台北晶华酒店 | Google |

```bash
cd places-agent
npx tsx --env-file=.env.local scripts/probe-t5-fill-review.ts hangzhou taipei
# writes tmp/probe-t5-fill-review.json; summary prints meal name + rating per day
```

## What to record

Per meal stop: `name`, `rating`, `user_ratings_total`, notes (`meal_low_signal` if any). Compare before vs after `agent-meal-116` rank.

## Baseline (before rank) — 2026-09-20

`npx tsx --env-file=.env.local scripts/probe-t5-fill-review.ts hangzhou taipei` · agent `:3010`.

| | Hangzhou | Taipei |
| --- | --- | --- |
| status / fill | ready 17/17 | ready 18/18 |
| elapsed | 59.9s | 198.6s |
| fill_s | 18.3 | 120.44 |
| trip_id | see `tmp/probe-t5-fill-review.json` | same |

**Hangzhou meals (AMAP):** 望江楼 4.5 · 西子宾馆牡丹厅 4.5 · 叶叶菩提 4.6 · 茶歌龙井 4.2 · **肯德基(西溪周家村店) 4.2** · 龙鼎楼 4.6. Reviews not on card (AMAP).

**Taipei meals (Google):** 台灣松屋 3.5 · 蔡記蚵嗲 4.2 · 光一一個時間 4.4 · 阿達師五星麵舖 4.1 · 春乃家 4.0 · 北投茶色 4.4. **`user_ratings_total` missing on cards** (mapper not yet writing `userRatingCount`).

Note: this baseline already avoids 2.2 canteens by chance; KFC remains a low-signal specialty miss. After rank: Google review floor + type exclude; AMAP still rating-only.

## After rank — 2026-09-20

`npx tsx --env-file=.env.local scripts/probe-t5-fill-review.ts hangzhou taipei` · agent `:3010` (restarted with meal-116). Artifacts: `tmp/probe-meal116-after.json`.

| | Hangzhou | Taipei |
| --- | --- | --- |
| status / fill | ready 17/17 | ready 15/15 |
| elapsed | 59.2s | 134s |
| fill_s | 17.57 | 69.34 |

**Hangzhou meals (AMAP):** 外婆家 4.6 · 香楼 4.7 · 叶叶菩提 4.6 · 龙井茂居 4.7 · 新荣记 4.9 · 洲际酒店杭州厅 4.7. No KFC; all ≥4.6; no `meal_low_signal`. Reviews still `—` (AMAP).

**Taipei meals (Google):** 晴天廚房 4.7 / **1599** · 貓之伊甸 4.9 / **252** · 青青食尚花園會館 4.7 / **12196** (lunch+dinner same day) · 高麗照茗茶 5.0 / **421** (both slots). **`user_ratings_total` now on cards** (mapper + fieldMask). All gated ratings; no `meal_low_signal`.

### Compare vs baseline

| Signal | Baseline | After |
| --- | --- | --- |
| Hangzhou min meal rating | 4.2 (KFC) | 4.6 |
| Taipei `user_ratings_total` | missing | present (hundreds–12k) |
| Taipei min meal rating | 3.5 | 4.7 |
| Contract gate (vitest) | n/a | TC-M116 Done |

Live probe is observational (LLM skeleton variance: TP fill count 18→15). Acceptance remains fixture TC-M116.
