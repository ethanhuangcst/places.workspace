Family backlog: [`product-backlog.md`](../../product-backlog.md)

---
title: 2play step-g chips vs tips 01 iconic lists
type: ops-lesson
status: active
as_of: 2026-09-05
tags:
  - iconic
  - fetch_trip_details
  - discover_places
  - plan_trip
  - travel_tips
  - adr-045
  - adr-046
  - adr-050
related_spec: ../../2play-specs/2play-design.md
related:
  - adr/ADR-045-iconic-places-unified-acquisition.md
  - adr/ADR-046-trip-store-pg-memory-fetch.md
  - adr/ADR-050-where2play-no-product-llm.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - agent/itinerary-ui-fetch-only.md
  - ../../agent-specs/real-agent-refactory.md
---

# Chips and tips 01 from plan_trip + fetch（Target）

**Target（2026-09-05，[ADR-050](../../adr/ADR-050-where2play-no-product-llm.md) / [real-agent-refactory](../../agent-specs/real-agent-refactory.md)）：**

- **第 6 题 / 必去芯片：** 目的地 geocode 成功后由 **`plan_trip`** 写入验真 `candidates`（`must_see`）；写卡时解析 `photos[0]`（[ADR-051](../../adr/ADR-051-discover-resolve-display-photo.md)）。UI **`fetch_trip_details` `fields: ["candidates"]`**。where2play **零 LLM**、不自行取图 — 不得本地模型列店名；空芯片不得伪造选项。
- **贴士四卡：** 目的地 + 起止日齐后由 **`plan_trip` 内** 写 `artifacts.tips` / `artifacts.visa`（一次 tips-prose；签证/天气事实先拉）。**01 必列必去** = 同一份验真 `must_see`，不是第二套 LLM 名单。展示只 fetch `artifacts`。2play **不**跑 tips-prose。
- 宿主不必另调 `discover_places` / `travel_tips` / Orizn MCP 拼芯片或四卡（过渡别名可指向同一内部函数）。

**As-built（至实现切片前，2026-09-02 合同）：**

- 芯片：`discover_places` 后热度打 `must_see` → fetch `candidates`。
- 四卡 01：`make_itinerary` 后 `travel_tips` → `artifacts.tips` → fetch。
- MCP 仅贴士、无行程时仍可调 `travel_tips`（ungrounded）；非 2play Plan 主路径 Target。

Ungrounded `findIconicPlaces` 仅历史/MCP 无池贴士路径；**不得**当可排程 / 第 6 题选项。
