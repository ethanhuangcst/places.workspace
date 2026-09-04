---
title: 2play step-g chips vs tips 01 iconic lists
type: ops-lesson
status: active
as_of: 2026-09-02
tags:
  - iconic
  - fetch_trip_details
  - discover_places
  - travel_tips
  - adr-045
  - adr-046
related_spec: 3.where2play/2play-specs/2play-design.md
related:
  - adr/ADR-045-iconic-places-unified-acquisition.md
  - adr/ADR-046-trip-store-pg-memory-fetch.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - agent/itinerary-ui-fetch-only.md
  - ../../1.places-agent/agent-specs/agent-design.md
---

# Step g chips come from discover pool; tips 01 from artifacts after skeleton

**As of 2026-09-02 host contract** (`agent-design.md` §23, `2play-design.md` §4.10):

- **助手步骤 g 芯片：** `discover_places` Phase A 类目搜建池后，内部 `findIconicPlaces` **只按热度给池内 POI 打** `must_see`（不另搜热点）。写入后 **`fetch_trip_details` `fields: ["candidates"]`**，取 `must_see` 名。点「规划行程」即开始 discover，与问答并行；步骤 g **等待**该 fetch。不要用 intake 期 ungrounded `travel_tips` 填芯片。
- **贴士四卡 01 必去地：** **`make_itinerary` 之后** 再写 `travel_tips`（优先带 skeleton）→ `artifacts.tips` → fetch `artifacts`。不要用写工具 HTTP 体渲染。

MCP 仅贴士、无行程时，仍可按 ADR-045 单独调 `travel_tips`（可 ungrounded）。那不是 2play Plan 主路径。

Ungrounded `findIconicPlaces`（`numDays >= 3` 一日游措辞）仅 **`travel_tips` 无池**路径。discover **不得**用无池 LLM 或二次搜点补芯片。

`travel_tips` 写路径：有 iconic 名则 HTTP 200 并双写，即使 tips-prose 失败。Fill 链不依赖贴士。
