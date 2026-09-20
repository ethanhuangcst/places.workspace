---
title: Skeleton bind meals vs fill — eval (Hangzhou / Taipei wall clock)
type: research
status: active
as_of: 2026-09-20
tags:
  - skeleton
  - fill
  - meals
  - performance
  - true-agent
related:
  - ../../adr/ADR-074-skeleton-meals-variant-a-only.md
  - ../../adr/ADR-049-verified-attraction-and-meal-slots.md
  - ../../adr/ADR-063-skeleton-only-plan-trip.md
  - ../maps/google-restaurant-search-latency.md
  - ../maps/fill-by-native-id-perf.md
---

# 骨架绑餐 vs 现 fill（评估）

**决策锁：** [ADR-074](../../adr/ADR-074-skeleton-meals-variant-a-only.md) **Proposed** — 若改 ADR-049 D3 只允许变体 A；否决 B；C 须显式对 ADR-063 取舍。本文是证据与墙钟，不是实施说明书。

## 现行 vs 提案

现行：骨架 = stay + 景点 + **餐档**；fill = 走廊搜餐 + Directions + 按需 Details。

提案：骨架 LLM 定起点+景点+餐厅并落 id；fill 只按 id 拉详情。LLM **不能编 native_id** — 须先 search/grounding（ADR-067 同构）。

## 变体 A / B / C（transit）

| 切法 | Fill | 真智能体 | 产品 |
| --- | --- | --- | --- |
| **A** | 仍 Directions；可省搜餐 | 符合 | T3 首屏变慢 |
| **B** | 只 Details | **否**（编 duration） | 假加速 |
| **C** | 几乎空 | 符合持环 | **毁掉** ADR-063 骨架先出 |

## 现行墙钟证据

### 杭州 3 日 · AMAP

| 阶段 | 来源 | 墙钟 |
| --- | --- | --- |
| Discover + make（无 fill） | signoff 2026-09-04 | 1.7s + make 13.2s |
| `plan_trip` skeleton_only | 2026-09-20 session | 22.9s；校验失败时可 3× make ≈ 120s |
| Fill 单站 | 2026-09-20 session | stay ~10ms；景点 0.6–0.9s；正餐 0.8–1.4s |
| Fill 整链粗算 | 同上 | ~12–18s |
| 可感知 | 骨架先出 | 首屏 ~15–25s；填完 ~35–45s |

低分食堂是 **距离第一、无评分闸**，不是少 fill LLM。正餐 **不是** 杭州墙钟主因。

### 台北 3 日 · Google（ADR-052 其他区）

| 阶段 | 来源 | 墙钟 |
| --- | --- | --- |
| Fill 整链 | fill-by-native-id-perf 2026-09-19 | **~60.8s / 20 站** |
| 正餐风险 | google-restaurant-search-latency | searchText × 走廊最多 6 次 × 25s + MCP；不可用杭州秒级外推 |

抄 id **不能**省 Directions，也 **不能**省正餐环搜（除非把搜挪到骨架 = A 的前移）。

## 估算（A 未实现，非 live）

- 杭州 A：总墙钟 **接近或略慢**；首屏因搜餐+更重校验 **变差**。
- 台北 A：Google 搜餐进骨架 → **首屏可接近今日 fill**。
- B 省腿但原则否决。C 首屏=填完。

## Live 基线刷新（2026-09-20，现行路径）

`places-agent` `npx tsx --env-file=.env.local scripts/probe-t5-fill-review.ts hangzhou taipei` · `skeleton_only: false` · agent `:3010` health 200。A/B/C **未实现**，本表只刷新对照。

| | Hangzhou 3d AMAP | Taipei 3d Google (TW, 晶华) |
| --- | --- | --- |
| trip_id | `cmu99lxnj000202twxfyjwncg` | `cmu99n496000a02twcfqo6f5y` |
| status / fill | ready **15/15** | ready **18/18** |
| **elapsed** | **55.2s** | **131.1s** |
| intake_s | 8.63 | 18.65 |
| origin_s | 0.18 | 2.17 |
| **skeleton_s** | **13.66** | **22.46** |
| **fill_s** | **15.43** | **67.81** |
| tips_s | 7.12 | 5.98 |
| total_s | 55.19 | 131.07 |

T3 骨架先出墙钟 ≈ intake + origin + skeleton（未含 fill）：杭州 **~22.5s**；台北 **~43.3s**。填完再加 fill：杭州 **~38s** 到 fill 结束（整包含 tips **55s**）；台北 fill 单独 **67.8s**（整包 **131s**）。

对照：杭州此前估算填完 ~35–45s（本趟 fill 15s 符合 12–18s 粗算；整包更长因 intake/tips）。台北 2026-09-19 fill-by-id spike **~60.8s / 20 站**；本趟 **fill_s 67.8s / 18 站**，量级一致，略慢。

结论不变：杭州餐搜不主导墙钟；台北 Google fill 才是大头。变体 A 把搜餐挪到骨架只会抬高台北 **skeleton_s / 首屏**，难压低 **total_s**。
