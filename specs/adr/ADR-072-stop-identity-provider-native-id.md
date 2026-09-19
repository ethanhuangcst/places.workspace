# ADR-072: 停点身份 = `(provider, native_id)`；Google 显示名跟 UI locale

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted**（2026-09-19）— 产品确认 1–12；实现故事 `agent-fill-113`。

**Does not supersede:** [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)、[ADR-051](./ADR-051-discover-resolve-display-photo.md)、[ADR-011](./ADR-011-hk-tw-independent-locales.md)。

**Amends:** [ADR-052](./ADR-052-map-provider-routing.md) D9 时机（见 D4）。

## Context

Lisbon 列表缺图（`Torre de Belém` / `Mosteiro dos Jerónimos`）曾用精确名、去音调子串、`sharedProperToken`、vendor search、`languageCode` 绑卡。根因是：

- Discover 池卡已有供应商 **`native_id`** 与 ADR-051 解析的 `photos[0]`。
- 骨架 LLM 候选行只有名字，停点常被译成葡语/中文，**指针未写上 stop**。
- `attachNativeIdsToSkeleton` 在 token 多命中（塔 vs Pastéis de Belém）时放弃打 id。
- Fill `matchCardByPointer` 无 id 则对名字；miss 后再用 **UI locale** 搜站名（CN + 葡语 query），与池英语名对不上。

Google Places 用什么 `languageCode` 搜，返回什么文案都可以；**同一地点的 `place_id` 不变**。高德 `poiid` 同理。身份不应走译名或词表。

## Decision

### D1 — 统一身份（Google 与 AMAP 同一套）

停点、池卡、filled 槽位的身份是 **`(provider, native_id)`**：

| 供应商 | `provider` | `native_id` |
| --- | --- | --- |
| Google | `GOOGLE_MAPS` | `places/ChIJ…` 或可解析的 Place id |
| 高德 | `AMAP` | POI id（如 `B0…` / `BV…`） |

附着、fill 抄图、对池求交、列表指针都只认这对字段。禁止为 Google 单开一套名字翻译逻辑。

### D2 — 骨架必须带池内指针

1. Attraction 候选进 LLM 时列出 **name + provider + native_id**。
2. 每个 attraction 停点必须抄一条池内指针；校验失败则 retry / 丢该站。
3. `attachNativeIdsToSkeleton`：已有合法指针则保留；否则仅 **名字与池卡完全一致** 时补指针。
4. **删除** 填站/附着出图路径上的模糊专名匹配（含把 `sharedProperToken` 当缩略图合同）。`sharedProperToken` 可留在 must_include / eligible 等非出图路径，直到另开故事删干净。

### D3 — Fill 按指针抄卡；无指针才 id 求交

1. 有 `(provider, native_id)` → 在 **同一 provider** 的池卡上查找 → 抄 `photos[0]`、坐标、deeplinks。
2. 无指针：向该供应商搜站名；结果里 **只留 native_id 已在池中的卡**。恰好 1 个则抄该池卡；0 或 >1 **不出图、不 `searched[0]`**。
3. 正餐仍按 ADR-049 现搜，不走景点池 id。

### D4 — 显示名：仅 Google 在 fill 写一次 UI 语言；AMAP 不译

- **不是** 把 POI 写入 i18n 目录（`CN.json` 等）。Layer A 仍是产品文案；地点专名是供应商 Layer B（ADR-011）。
- **Google：** fill 时 `get_place_details({ native_id, locale: UI })`（`languageCode` 来自 `LOCALE_LANG`）。将返回的 `name` **写一次** 到 `stop_display.stop.name`。若该 hop 已为补图打 Details，顺带取名，不另开真源。失败则保留骨架/池原名。
- **AMAP：** 保持池/搜索原名，不加语言翻译步。
- **相对 ADR-052 D9：** 跨文脚本改名允许发生在 **fill 写槽位那一次**（Google only）。用户打开 place sheet 时仍不得用另一种文脚本盖掉槽位名（避免列表拉丁、详情突然变中文的闪变）。列表与 sheet 应同为 fill 已写的 UI 语言 Google 名。

### D5 — 性能预期（非本故事的加速目标）

按 id fill **不是** 把 3 天填站从 ~50–60s 打到十几秒的手段。墙钟仍由 **每站 Directions × 3（walk/transit/drive）** 和 **正餐搜店** 主导。

Live 量级（2026-09-19 spike，locale CN，3 天）：台北 ~61s / 20 站；里斯本 ~49s / 22 站；stay 已 ~15ms；景点 hop ~0.4–1.5s。

按 id 去掉景点补搜/geocode/补图 Details：乐观（几乎每站译名 miss）整趟约 **10–25%**；仅两站 miss 则 **&lt;5%**。验收写成「有指针则图从池抄、不再因译名补搜」。不得把「填站显著变快」写进本故事 AC。再加速 Directions/餐搜另开故事。

### D6 — 禁止项

- 城市/景点葡英中对照表、cognate、为常错名词加场馆词表来出图（ADR-042）。
- 用 UI `languageCode` 当身份匹配器（搜中文去对英语池名）。
- 2play 复制模糊别名（ADR-010）；BFF 只渲染槽位 `name` / `photos[0]` / 指针。

## Rationale

- 供应商 id 跨语言稳定；译名不稳定。
- 词表会再次把目的地知识写进源码（ADR-042 第四次 cognate 已否）。
- Google `languageCode` 适合 **展示**（D10），不适合 **身份**。
- 高德中文池无需再译。

## Consequences

- 实现面：`buildSkeletonUserMessage` / `attachNativeIdsToSkeleton` / `matchCardByPointer` / fill 抄卡；单测 CN locale + 葡语站名 + 英语池同 `ChIJ`；`no-city-hardcode` 保持绿。
- 故事：`agent-fill-113`（places-agent）；2play 不复制别名。
- 知识：[`knowledge/maps/fill-by-native-id-perf.md`](../knowledge/maps/fill-by-native-id-perf.md)。

## Date

2026-09-19
