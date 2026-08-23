---
title: Lisbon discover A/B probe — deterministic vs LLM queries
type: ops-lesson
status: active
as_of: 2026-08-22
tags:
  - places-agent
  - discover_places
related:
  - ./discover-xian-ab-probe.md
  - ../../adr/ADR-038-discover-places-quality.md
---

# Lisbon 3-day discover A/B probe (2026-08-22)

```bash
cd 1.places-agent && python3 scripts/probe-lisbon-discover-ab.py --l2
# JSON: tmp/probe-lisbon-discover-ab.json
```

Providers: `GOOGLE_MAPS` only. Locale EN.

## L1

| | baseline discover | armA seeds | armB LLM queries |
| --- | --- | --- | --- |
| must_see (Belém+Jerónimos+São Jorge+Alfama/Baixa) | fail (no Jerónimos / Alfama) | **pass** | **pass** |
| dining local signal | 0 (brunch/coffee heavy) | 17 | 22 |
| elapsed | 0.9s | 4.5s | ~13s (LLM 8.5 + search 4.4) |

Arm B queries were close to Arm A (Belém Tower, Jerónimos, Alfama, São Jorge, Tram 28, Pastéis de Belém, bacalhau, sardines) plus fado / Cervejaria Ramiro.

## L2 (direct OPENAI_CN day arrange)

Both arms day 1 packed classic icons. Arm A day 3 drifted to pastry crawl; Arm B day 2–3 leaned fado + viewpoints. Neither needs L1 LLM for Lisbon must-sees when seeds exist.

## Verdict

**历史结论（2026-08-22，已过时为产品方向）：** 曾写「ship deterministic Arm A catalog/seeds」。  
**更正（2026-08-23，[ADR-042](../../adr/ADR-042-no-city-encyclopedia-in-source.md)）：** Arm A 只证明「定向 query 有效」，**不**证明应按城维护源码 CATALOG。里斯本种子未合入生产是正确的产品判断；下一步应是目的地无关热门信号（或外置 pack），而非把 Belém 写进 `discover-must-see.ts`。LLM query arm 仍不作 L1 默认。

Baseline discover without a destination-agnostic popularity path still misses Jerónimos/Alfama-class coverage — that gap remains open under ADR-042.
