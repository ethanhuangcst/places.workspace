# where2play plan L2 — OPENAI_CN + Mode H (ADR-037)

**Date:** 2026-08-23  
**Related:** [ADR-037](../../adr/ADR-037-where2play-plan-l2-quanzil.md), [ADR-036](../../adr/ADR-036-where2play-assistant-quanzil.md), [ADR-038](../../adr/ADR-038-discover-places-quality.md)

## Target pipeline (Mode H)

```text
L1  discover_places     places-agent   候选池（ADR-038 seed / 过滤）
L2  arrange_day host    places-agent   system_prompt + user_prompt（无 LLM）
L2  OPENAI_CN execute     where2play     结构化日行程 JSON
UI  NDJSON progressive  where2play     slot_preview → slot → day_done
```

与 MCP ChatBox / Cursor **同源** prompt（agent Feature **35** Done）。

## As-built vs MVP-3

| 层 | MVP-2 as-built | MVP-3 target |
| --- | --- | --- |
| L2 prompt | BFF 本地 `buildArrangeDayMessages` | agent `execution=host` |
| L2 LLM | 2play OPENAI_CN | 不变 |
| Transit | 估时合成、易默认 ~15min | **plan-13** `legs_to_here` |
| 地标 | 不稳定（本地 prompt + discover 截断） | Mode H + ADR-038 + E2E must-see |

## 禁止

- 主路径 `arrange_day` **execution=agent**（agent 内等满 JSON）
- `plan_itinerary` 作为 2play Plan 默认

## MVP 批次

- **MVP-3** features **31–33**；E2E `make test-e2e-mvp3-live`
- Chat → **MVP-4**；Replan → **MVP-5**（复用 MVP-3 管线）

## Links

- [`2play-stories.md`](../../../3.where2play/2play-specs/2play-stories.md) · [`itinerary-design.md`](../../../3.where2play/2play-specs/itinerary-design.md)

## Sync note (2026-08-23)

As-built dual-channel contract consolidated in [ADR-043](../../adr/ADR-043-chatbox-mcp-and-cross-product-closure.md): 2play = Mode H + enrich; ChatBox MCP = force agent. Prefer that ADR over older “target Mode H / plan-13 未做” wording.
