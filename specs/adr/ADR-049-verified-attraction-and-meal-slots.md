# ADR-049: 可规划景点门槛 + 骨架餐档 + 目的地景点库最后做

## Status
Accepted（2026-09-04）

## Context

杭州 3 日 UI 路径：`discover` 池不空，`make_itinerary` 仍 HTTP 502、骨架空。根因不是 ADR-048 的错误城市锚点，而是：

1. 供应商把**合称 / 风景名胜区**（如「西湖十景」「杭州西湖风景名胜区」）当成可排景点；芯片与 `must_include` 覆盖用**子串**对集合名。
2. 骨架 LLM 把**餐馆名**写进日框架，与景点覆盖、验证硬闸缠在一起。
3. 若先建「目的地景点库」再止血：正确性仍依赖同一套门槛；库只解决**跨行程复用**，不解决当次脏名进规划。

[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) 禁止的是**源码按城百科**（`CATALOG` / if-城），**不是**运行时、按供应商 `place_id` 累积的验证登记表。

相关：[ADR-038](./ADR-038-discover-places-quality.md)、[ADR-045](./ADR-045-iconic-places-unified-acquisition.md)、[ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)、[ADR-048](./ADR-048-skeleton-geo-anchor-is-destination.md)。

## Decision

1. **可规划景点（eligible attraction）是共用谓词**，discover 写入 `candidates`、芯片、`findIconicPlaces` 打标、`make_itinerary` 入模池、将来跨行程入库**同一实现**。默认门槛（L0，不另打 `get_place_details`）：有稳定供应商 `native_id`（或等价 place id）、有坐标、名称不是合称/集合/「…名胜区」类区域、类型为景点而非餐馆。不扩 CATALOG。
2. **`must_include` 对不上 eligible 点时降级**（忽略该项 + 可观测 warning），**不得**因此整单 502。用户仍可用芯片点具体景点。
3. **`make_itinerary` 只排景点顺序 + 每日餐档（slot）**，不把具体餐馆写入骨架。餐馆在 **`plan_next_stop` 填站时**按相邻景点再搜再落点。不恢复 / 不新开 `plan_day_trip`。
4. **餐馆不进目的地景点库。** 餐点是行程填充产物，不是目的地登记对象。
5. **目的地景点库（Destination + Poi registry）是最低优先级、最后实现。** 身份 = 目的地 geocode `place_id` + 景点 `native_id` + 别名，不是模糊中文名。L1 详情异步、不挡 make。没有库时，每次仍现搜 + 当次过滤；正确性不依赖库。
6. **Trip + `fetch_trip_details` 仍是行程真源**（ADR-046）。库是跨行程景点登记，不是第二套行程。
7. **双门槛：** 主闸 = 决策 1（discover 入库谓词）。安全阀 = `make_itinerary` / `plan_next_stop` **再用同一谓词**扫池。仅**漏网脏卡**（无 id、无坐标、合称/名胜区、误标餐馆）才修订本 trip 的 `candidates`。缺电话/营业时间/照片 = L0 预期，**不是**脏数据，不得因此改池。
8. **内部 `patchTrip` 取代 `patch_skeleton`（修订 ADR-046 D8/D9 的方法名与字段面，不改「仅内部」）。** 单数 core；一次乐观锁 `revision`；声明式补丁可及 `candidates` / `skeleton` / `filled` / `constraints` / `artifacts`（remove / replace / set-slot），禁止整本覆盖。调用方只是现有写工具。 **不**暴露 HTTP/MCP，**不**新增 `patch_trips` / `POST /v1/patch_trip`。不得当第二套 discover 或 details 验真。

## Rationale

| 备选 | 为何不选 |
| --- | --- |
| 先建库再规划 | 库不能单独去掉合称/502；拖长止血 |
| 扩 CATALOG / 杭州行 | 违反 ADR-042 |
| 保留骨架绑餐馆 | 继续与 must_include / 覆盖校验耦合 |
| 新 `plan_day_trip` 整天 LLM | 与轻骨架 + `plan_next_stop` 重复 |
| 把库当产品 must-see 源码表 | 与 ADR-042 混淆；库必须是运行时 upsert，不是 TS 百科 |
| 读池失败 = 脏 → 对外 patch | 缺 L1 字段不是脏；万能 HTTP patch 与 ADR-046「禁止暴露 patch」冲突 |

## Consequences

- 实现顺序固定为 **S1 门槛+降级+内部 patchTrip → S2 餐档骨架 → S3 填站搜餐 → S4 景点库**（见 agent `0.refactor-plan` MVP-22）。不得为「先有库」推迟 S1。
- S4 开工时不得重写门槛；只调用 S1 谓词做 upsert。安全阀修的是**本 trip `candidates`**，不是跨行程库。
- 2play 芯片与助手文案只展示 eligible 景点；餐档展示为「午餐/晚餐」而非店名，直到 fill。
- 热门城市第一次规划仍付当次搜索成本，直到 S4。
- `patch_skeleton` 实现应迁到 `patchTrip`；对外工具清单不变。

## Date
2026-09-04
