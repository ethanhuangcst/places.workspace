---
title: ADR-052 已决却未执行 — discover 扩源漂移
type: ops-lesson
status: active
as_of: 2026-09-06
tags:
  - maps
  - adr-052
  - discover
related_spec: workspace-specs/adr/ADR-052-map-provider-routing.md
related:
  - adr/ADR-052-map-provider-routing.md
  - knowledge/maps/vendor-adapters.md
  - knowledge/agent/places-agent-loop.md
---

# ADR-052 已决却未执行 — discover 扩源漂移

## Summary

地图路由已整合进 ADR-052（大陆 AMAP-only，仅 0 卡再 Google），但杭州/西安行程景点仍 AMAP+Google 混编。根因不是 resolver 判错区，而是 **discover 主路径从未删掉 Feature 34 的无条件双源扩源**，且 ADR 初稿 D2 留了一句「若产品需要可扩源」，把旧代码合法化了。

## Why the accepted ADR was not followed

1. **两套实现，只改了下层。** `resolveProviderStrategy` 与 `shouldTryGoogleAfterEmptyAmap` 按 D2/D4 写对。`searchCandidatePools` 不直接用 strategy，而是经 `resolveDiscoverProviders`：strategy 给出 `["AMAP"]` 后立刻改成 `["AMAP","GOOGLE_MAPS"]`。2play 省略 `providers[]` 之后，**必走**这段扩源。
2. **ADR 自相矛盾被当成许可。** 初稿 D2 写「Discover L1 若需要大陆双源，可在门面内扩源」；D4 / Rationale 又否决「大陆默认双源」。实现把「若」做成 always-on，评审只盯 2play 不再拼 `providers[]`，没删 agent 门面扩源。
3. **Feature 34 质量故事压过路由故事。** Arm A 用 Google RELEVANCE + 双源「补 must-see 覆盖」。ADR-052 后没有把 Feature 34 标为部分作废，测试仍默认双源 jobs。
4. **表面分层未写进 ADR。** 列表用槽位中文名；详情 `get_place_details(slot.provider)`。槽位已是 Google 时，详情半秒变英文（Google getDetails 未带 `languageCode`）。看起来像「详情改了供应商」，实际是 **框架层已经写错 provenance**。
5. **Directions 默认列表未改。** 省略 `providers[]` 时硬编码 `["GOOGLE_MAPS","AMAP"]`，违反 D7「用已解析列表」。

## Evidence

- 杭州「杭州植物园」：列表中文；place sheet 先中文后英文；供应商 Google。
- `itinerary-planner.ts` `resolveDiscoverProviders`（Feature 34 Arm A 注释）。
- `google/direct.ts` `getDetails` 无 `languageCode`；`searchText` 有。
- `place-sheet.tsx`：`title = details?.name ?? slot.name`。

## Lesson / guidance

- 路由 ADR 的验收必须以 **discover / fill 产出卡的 provider 分布** 为准，不能只测 `provider-resolver` 单测和 2play 省略 `providers[]`。
- ADR 禁止写「若产品需要」却不设默认关闭；例外必须有显式 flag，否则旧代码会留下。
- **Fixed (Feature 89):** 删扩源 → Directions 走 strategy → getDetails 带 locale → sheet 不覆盖 CJK 槽位名。
- **Hangzhou follow-up (2026-09-06):** 大陆 AMAP-only 后骨架失败，不是路由回退，而是 (1) AMAP 把西湖子点命名为 `…风景名胜区-子点`，资格过滤整段丢弃；(2) CN QLP 欧式复合词（城堡/宫殿）对高德空结果。修复：`unwrapScenicChildName` + 短 CN 模板（景点/公园/博物馆/寺庙）。用**新**杭州行程验收。
- 见 ADR-052 D9/D10。

## Links

- [ADR-052](../../adr/ADR-052-map-provider-routing.md)
- [vendor-adapters.md](./vendor-adapters.md)
