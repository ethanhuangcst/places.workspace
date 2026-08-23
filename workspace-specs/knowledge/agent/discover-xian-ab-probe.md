---
title: Xi'an discover A/B probe — deterministic L1 vs LLM queries
type: ops-lesson
status: active
as_of: 2026-08-22
tags:
  - places-agent
  - discover_places
  - where2play
related_spec: workspace-specs/adr/ADR-038-discover-places-quality.md
related:
  - ./discover-places-quality-seed-filter.md
  - ../../adr/ADR-038-discover-places-quality.md
---

# Xi'an discover A/B probe (2026-08-22)

## How to reproduce

```bash
cd 1.places-agent
python3 scripts/probe-xian-discover-ab.py --l2
# JSON: tmp/probe-xian-discover-ab.json
```

Arms: **baseline** = live `discover_places` (no `providers`); **armA** = dual-provider catalog seeds + dining rank (no LLM); **armB** = OPENAI_CN generates search queries only → same `search_*` + filter/dedupe/cap.

## L1 scorecard

| Metric | baseline | armA (deterministic) | armB (live LLM queries) |
| --- | --- | --- | --- |
| must_see_ok (陵+雁塔+城墙+钟鼓) | no | **yes** | no (缺雁塔类进池) |
| terracotta | — | 秦始皇陵 | 秦始皇陵 |
| dayan | 大雁塔 | 大雁塔南广场 | — |
| wall / bell | 城墙 / — | 城墙 / 钟楼 | 城墙 / 钟楼 |
| wall_count ≤2 / fragments | ok / 0 | ok / 0 | ok / 0 |
| dining local / chain / far | 4 / 0 / **10** | **24** / 0 / **0** | **24** / 0 / **0** |
| elapsed / search calls | 0.5s / discover×1 | 7.3s / 24 | 15.0s (LLM 9s + search 6s) / 24 |

**Arm B LLM queries (live):**  
景点 `秦始皇帝陵博物院, 陕西历史博物馆, 西安城墙, 西安钟楼, 大雁塔, 大唐不夜城`；餐 `羊肉泡馍, 肉夹馍, biangbiang面, 凉皮, 臊子面, 甑糕`。  
有「大雁塔」query 仍未进最终池（过滤/vendor 命中形态），故 must_see_ok=false。

## L2 (agent `arrange_day`，同池对比)

| | armA | armB |
| --- | --- | --- |
| 墙钟 | ~172s | ~118s |
| Day1 | 陕历博 · 钟楼 · 城墙 + 本地餐 | 陕历博 · 钟楼 · 城墙 + 餐 |
| Day2 | 博物院 · **大雁塔南广场** + 餐 | 博物院 · 小雁塔 + 餐（**无陵/无雁塔主点**） |
| Day3 | 小雁塔 · 陕历博综合楼 + 回民街 | 青龙寺 · 半坡 · 世博园（**偏博物馆堆**） |
| 池内秦始皇陵是否排进三日 | **否**（仍在候选，L2 未选） | **否** |

交通：两臂均无结构化 leg（本探针范围外）。

**Verdict**

1. **L1 优先落地 Arm A（确定性）**：双源 seed + 馆名优先 + 本地餐加权，已稳定过 must-see 门禁；baseline（大陆默认 AMAP-only discover）仍失败。  
2. **不必为 L1 接 LLM**：Arm B live 更慢，且本次未优于 A；LLM 写 query 不能保证 vendor/过滤后必去进池。  
3. **下一刀在 L2 / 池噪声**：两臂 L2 都可漏掉池内「秦始皇陵」；Arm A 池仍含「菜鸟驿站/修脚房」等噪声 → 需硬 must-see 覆盖 + 更严 deny，而非给 discover 接 LLM。  
4. **ADR-038「L1 无 LLM」维持**；产品故事：把 Arm A 脚本策略 merge 进 `discover-must-see` / `searchCandidatePools`，并修 mainland 默认仅 AMAP。

**落地（2026-08-23）：** Feature **34–38** 已合入主路径（Arm A 种子/双源/餐排、硬必去、Mode H、`legs_to_here`、MCP session）。本探针仍可用于回归对照。

## Caveats

- Agent `arrange_day` 对环境 TLS/OPENAI_CN 敏感；探针 L2 在本次重测中走通 `agent_arrange_day`。  
- OPENAI_CN 证书常为 IP-SAN；探针 Arm B 经 Node `NODE_TLS_REJECT_UNAUTHORIZED=0` 调用（仅脚本）。
