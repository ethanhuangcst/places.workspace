# ADR-052: 地图供应商路由（整合）

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted** — 地图「搜哪家 / 怎么判区 / 何时回退 / 运输层 / 打开哪张图」的单一规范。

**Supersedes:** [ADR-005](./ADR-005-caller-driven-providers.md)、[ADR-006](./ADR-006-provenance-client-nav.md)、[ADR-017](./ADR-017-gmaps-mcp-fallback.md)、[ADR-026](./ADR-026-region-based-provider-auto-selection.md)、[ADR-030](./ADR-030-geocode-first-region-detection.md)、[ADR-031](./ADR-031-amap-empty-google-fallback.md)。

**Does not supersede:** [ADR-014](./ADR-014-open-meteo-weather.md)（天气不是地图供应商）、[ADR-022](./ADR-022-timed-itinerary.md)（时刻行程合同；其中 Directions 用哪家服从本 ADR）、[ADR-037](./ADR-037-where2play-plan-l2-quanzil.md)（2play 不持地图 key）、[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)、[ADR-048](./ADR-048-skeleton-geo-anchor-is-destination.md)、[ADR-051](./ADR-051-discover-resolve-display-photo.md)。

## Context

供应商路由曾拆成多条 ADR，后文互相覆盖、调用方实现漂移：

| 旧 ADR | 当时决定 | 为何不够单独当现行规范 |
|--------|----------|------------------------|
| 005 | 调用方传 `providers[]`；agent 不按大陆强塞 AMAP | 调用方自写 `providersForPin` 双源 → 北京搜到美国店 |
| 026 | 省略 `providers[]` 时按区域自动选；大陆 AMAP-only | 检测含 CJK 占比，把港台日韩当大陆 |
| 030 | 坐标 / Geocode 国家文本优先；删除 CJK 启发式 | 未写清应用不得再按汉字组列表 |
| 031 | 自动 AMAP-only 且 0 卡 → 再搜一次 Google | 与「大陆双源并行」易被混为一谈 |
| 006 | 结果带 provenance；**打开地图**按客户端环境 | 常被误当成搜索路由 |
| 017 | Google 直连失败 → Worker MCP；不改成 AMAP | 运输层，不是第三家供应商 |

2026-09 确认：2play 按 CJK 强制 `["AMAP","GOOGLE_MAPS"]`、discover 把 AMAP-only 扩成双源并污染酒店搜，违反 026/030，且放大「店名含钟楼 → 景点卡」类错绑。

## Decision

### D1. 谁决定 `providers[]`

1. **默认：** 调用方（what2eat、where2play、chat 工具环）**省略** `providers[]`，传 `address` 和/或 `near`。places-agent `resolveProviderStrategy` 选定供应商后再 `fanOut`。
2. **覆盖：** 调用方**显式** `providers[]` 一律生效（调试、强制单源）。显式列表**不**触发 D4 空结果回退。
3. **禁止：** 应用按汉字占比、locale、或「大陆双源」表自己拼 `providers[]`（含 `providersForDestinationText` / `providersForPin` 对大陆传 AMAP+Google）。
4. **Chat：** 工具环剥掉模型填的 `providers[]`，走自动选择。HTTP 直调不剥。
5. **2play / what2eat 不持** `AMAP_*` / `GOOGLE_MAPS_*`（ADR-037）。

### D2. 三区域 → 搜索供应商

| 区域 | `searchProviders` | enrich |
|------|-------------------|--------|
| **大陆** | `AMAP` only | — |
| **香港** | `GOOGLE_MAPS` + `AMAP` | Tripadvisor |
| **其他**（含台湾、海外） | `GOOGLE_MAPS` | Tripadvisor |

台湾排除 AMAP（覆盖差）。Locale（EN/CN/HK/TW）**不**决定供应商：上海的 EN 用户仍 AMAP；东京的 CN 用户仍 Google。

**Discover / 骨架 / fill 与上表同一套**（2026-09-06 修订）：**废除**「Discover L1 可在门面内把大陆 AMAP-only 扩成 AMAP+Google」的例外。`discover_places`、`make_itinerary` 候选池、`plan_next_stop` 搜餐/补搜，一律走 `resolveProviderStrategy` + D4，**禁止** `resolveDiscoverProviders` 式无条件扩源。Stay / 酒店搜同样 D2 + D4（见 [ADR-053](./ADR-053-origin-stay-as-stop-card.md)）。

### D3. 区域怎么判（030，废止 CJK 占比）

优先级：

1. 调用方已给 **`near` 坐标** → 边界框（台湾排除 → 香港框 → 大陆框）。不调 Geocode。
2. 仅有地址文本 → **Google Geocode**，用 `formatted_address` 的国家/地区（China / Hong Kong / Japan…）。
3. **地址文本优先于** geocode 坐标（中国框与韩蒙日重叠）。
4. Geocode 失败 → 城市/地区 **marker 列表**（繁简都要收录）。
5. 仍不明 → **其他**（Google）。未知 CJK **不是**大陆。

**禁止**用 CJK 字符占比当大陆信号。

### D4. 大陆 Google 何时上场（2026-09-07 修订：禁用回退）

**大陆（Mainland China）景点、餐厅、交通一律禁用 Google 回退。**

仅当以下**全部**满足才允许 Google 上场：

- 区域**不是**大陆（香港、台湾、海外），且
- 该次 `search_places` / `search_restaurants` 在该区域的默认供应商下 **0 张卡**（仅适用于非大陆区域）。

**禁止：**

- 大陆默认双源并行（含 discover / QLP jobs 同时打 AMAP 与 Google）。
- 大陆 AMAP 0 卡后再搜 Google（**原 D4 回退条款废止**）。
- AMAP **报错**当空结果。
- 显式 `["AMAP"]` 再回退 Google。
- Directions 在大陆因 AMAP 失败而回退 Google。

**理由（2026-09-07）：** 实测 Google 在大陆数据不准确（景点重复、餐厅信息陈旧、交通时长错误）。即使 Google 有数据也不可信，回退会引入更差的结果而非兜底。AMAP 0 卡时应当标记为 `partial` 并向用户暴露失败，而不是静默回退到更差的 Google。

**Discover 不是 D4 的例外。** 大陆 attraction 模板只组 AMAP jobs；AMAP 0 卡时不再扩源。不得先双源再按评分混排进池。

### D5. Google 运输层（原 017）

`GOOGLE_MAPS` = Google 数据。先直连 `maps.googleapis.com`（`GOOGLE_MAPS_API_KEY`）；仅出网失败再 Cloudflare Worker MCP（`GMAPS_MCP_*`，先 `tools/list`）。`sources[]` 仍为 `GOOGLE_MAPS`，没有 `GMAPS_MCP` 供应商。Worker 不是 places-agent `/mcp`。直连失败 **不**改成 AMAP，除非调用方本来就要 AMAP。

### D6. 打开地图 ≠ 搜哪家（原 006）

结果带 `provider` / `sources[]` / 无密钥 deeplink。UI 按**客户端环境**选打开哪条链（大陆机可优先 AMAP 链）。不因此改搜索供应商。

### D7. Directions（约束 ADR-022 §5）

用**已解析**的 `providers[]` 算路。省略时先 `resolveProviderStrategy`（大陆 AMAP），**禁止**硬编码默认 `["GOOGLE_MAPS","AMAP"]`。列表里已有 AMAP 且 locale 为 CN/HK/TW 时：先 AMAP Directions，再 Google 补（**大陆除外**：大陆 AMAP 失败不回退 Google，标记 `partial`）。**不**因 locale 把 AMAP 注入未请求的列表。失败不编造时长；`skipped` 写明供应商。

**2026-09-07 修订：** 大陆 Directions 一律 AMAP。AMAP transit 调用必须传 `city` 参数（否则 AMAP 默认上海，导致非上海城市算路错误）。AMAP 失败 → `transit_outcome: "partial"`，不回退 Google。

### D8. 相邻硬闸（不在本 ADR 重开）

- 天气：仅 Open-Meteo（ADR-014）。
- 骨架 80km 锚点是**城市**，不是酒店 geocode；intake 用 `search_places(query, address=目的地)`（ADR-048）。命中后整卡与 fill 只抄：[ADR-053](./ADR-053-origin-stay-as-stop-card.md)。
- 不按城写死供应商或 POI 表（ADR-042）。
- 可展示图在 agent 解析；2play 不代理 Photo（ADR-051）。

### D9. 行程三层表面用同一身份（2026-09-06）

供应商在 **search / discover 写入 PlaceCard** 时钉死。之后只抄卡，不换源。

| 表面 | 规则 |
|------|------|
| **行程框架**（discover 池、骨架、fill） | 省略 `providers[]`。大陆候选 `provider` 应为 `AMAP`（除非该次搜索走了 D4 且 AMAP 0 卡）。Fill **拷贝**池内卡；禁止跨供应商按 rating 重选「同一景点」。合并不把 Google 英文名当成另一张必去卡。 |
| **行程详情 / 列表** | 渲染槽位上的 `name`、地址、`provider`、`native_id`、`photos[0]`。列表 **不**调用 `get_place_details`、**不**重搜。 |
| **Stop 详情**（place sheet） | `get_place_details({ provider: slot.provider, native_id: slot.native_id, locale })`。只打 **槽位这一家**，禁止 fan-out 第二家。Google 详情必须带 UI `languageCode`（与搜索一致）。加载前显示槽位中文名；详情返回后 **不得**用另一种文脚本盖掉槽位已有 CJK 名/地址（拉丁文详情名不覆盖「杭州植物园」）。供应商徽标保持槽位 provenance。 |

旧行程若槽位已是 `GOOGLE_MAPS`，详情仍走 Google（可带中文 language）；**新**大陆行程不得再写入 Google 景点卡，除非 D4。

### D10. 详情语言跟随 UI locale，不跟随供应商默认英文

`get_place_details` 把 `locale` 传进 adapter。Google Places Details 请求带 `languageCode`；AMAP 详情用中文/对应语言参数。缺语言导致「列表中文、详情半秒变英文」视为缺陷，不是可接受的 provenance 展示。

## Rationale

- 大陆并行 Google 会对中文地址返回美国结果（what2eat 北京案例）。
- CJK ≠ 大陆（中環、銀座、明洞、臺北）。
- 空 AMAP 再一次 Google 保住错字可发现性，又不把双源变成默认。
- 运输失败换通道，不换数据品牌；打开 App 是客户端问题。
- 一条 ADR 避免 005「永不强塞」与 026「大陆自动 AMAP」并读时的误解。

**否决：** locale 路由；大陆默认双源（含 Feature 34 / `resolveDiscoverProviders` 无条件扩源）；用 CJK 判区；Google 出网失败改 AMAP；2play/BFF 维护第二套区域表；discover 扩源后污染 stay **或** 行程景点池。

## Consequences

- 现行实现：`provider-resolver.ts`、`tools.ts`（`shouldTryGoogleAfterEmptyAmap` **2026-09-07 起一律返回 `false`**）、`resolveDiscoverProviders`（**不**扩源）、`resolvedDirectionProviders`（D7）、Google `direct` getDetails 带 `languageCode`。
- **2026-09-07 变更：**
  - `shouldTryGoogleAfterEmptyAmap` 无条件返回 `false`（大陆禁用 Google 回退）。
  - `ItineraryPreferences` 新增 `drive_preferred`（自驾/打车优先），与 `transit_preferred` 并列。
  - `transitPreferred` / `drivePreferred` 正则增加 "交通"（匹配 "公共交通+步行"）。
  - AMAP Directions 调用传 `city` 参数（修复非上海城市默认上海导致算路错误）。
  - Transit 坐标匹配优先 `native_id` → 精确名 → 归一化名；失败标记 `transit_outcome: "partial"`。
  - `ItineraryTransitSlot` 扩展 `from` / `to` / `legs[]` / `outcome`，行程详情页结构化渲染交通药丸。
  - `attractionClusterKey` 前缀族聚类：剥离 "景区"/"售票处"/"重建记" 等卫星后缀，避免同一地标重复推荐。
  - `nominateMustSeeViaLlm`：LLM 提名 3-5 必去点（含 day-trip），再 `search_places` 落地坐标。
- 2play：省略 `providers[]`；place sheet 服从 D9（CJK 槽位名优先）。Feature **89** Done。
- 旧 ADR 仅作历史；新工作只引本文件。
- 知识文若仍写「CJK>30% → 大陆」、「Discover 可扩双源」或「ADR-005 禁止自动 AMAP」，以本 ADR 为准。
- 漂移原因与修复：[knowledge/maps/adr-052-discover-expansion-drift.md](../knowledge/maps/adr-052-discover-expansion-drift.md)。

## Date

2026-09-06（原版）
2026-09-07（修订：大陆禁用 Google 回退、drive_preferred、AMAP city 参数、坐标匹配、交通 UI 结构化、前缀族去重、LLM 必去提名）
