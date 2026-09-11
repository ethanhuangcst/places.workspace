# ADR-065 — 必去提名 vs 建骨架 关系（B+C）

- **Status:** Accepted（as-built T3+ geo gate）· discovery / nominate-vs-skeleton relationship **superseded by** [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md)
- **Date:** 2026-09-10
- **Deciders:** product + agent
- **Related:** [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) · [ADR-050](./ADR-050-where2play-no-product-llm.md) · [ADR-060](./ADR-060-intake-no-forced-nominate.md) · [ADR-062](./ADR-062-mvp-t3-skeleton-vs-t4-nominate.md) · [ADR-063](./ADR-063-skeleton-only-plan-trip.md) · [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md)
- **Design:** [`../agent-specs/agent-design.md`](../agent-specs/agent-design.md) 「能力逻辑与提示组合（内部意图）」

## Context

MVP-T3 `skeleton_only` 不跑必去提名（ADR-062），候选池仅靠通用模板 query 扩展，骨架 LLM 倾向只排市中心，远郊一日游簇（如 Lisbon 的 Sintra/Cascais/Roca）排不进。需要决定**必去提名**与**建骨架**两项内部能力的关系，并解决 T3 远簇成日质量，同时守 TRUE AGENT 原则与 ADR-042（无城市 POI 百科）。

TRUE AGENT 原则：循环在 places-agent 进程内；模型看「行程目标 + 已有 Trip + 可用工具」决定 act 或停；每项能力是可被模型独立 act-or-stop 的内部意图；事实闸由代码执行；能力**不**是对外 API，**不**注册 MCP 工具。

## Decision

采用 **B+C**。

### B — 提名独立能力，骨架当偏好消费

- 必去提名是**独立内部意图**：LLM 产短地名 + 一句理由（不排行程、不出坐标）→ 代码 `groundNominatedName`/`hydrateNominatedCard` 落到真实 PlaceCard → 打 `must_see` 并入池；落不下丢弃。
- 建骨架是**独立内部意图**：`buildSkeletonUserMessage` 在候选行打 `[must-see]`，模型当偏好排；`validateSkeleton` 事实闸不变（池归属、must_include 覆盖、pace、meal、跨日唯一、area alias 剔除）。
- 契约：提名产物 = `candidates[].must_see=true` + 理由文本（供 T4 助手展示与 chat refine）；骨架以偏好消费，**不**改 validator。

### C — 代码侧 geo 多样性闸（目的地无关）

- 给 `make_itinerary` 增加一道**目的地无关**的 geo 多样性闸：haversine 聚类 / 远簇独立成日 / 复用 `trimThemedDayOutliers`。
- 让 T3 `skeleton_only`（不跑提名）也能把远郊一日游簇排成独立日。
- **不**按城市名查表；**不**引入城市 POI CATALOG（ADR-042）。仅用候选坐标 + 锚点 haversine。

## Why B+C over alternatives

| 选项 | 结论 |
| --- | --- |
| A 合并提名入骨架提示、未来 sunset 提名 | 拒绝。合并两意图为一次 LLM 调用 → 模型失去独立 act-or-stop；骨架池归属事实闸被提名推理污染；sunset 与 TRUE AGENT 相悖；T3 重开提名违反 ADR-062 |
| B only | 可，但 T3 仍缺远簇成日能力 |
| D 等 T4 提名再修 | 拒绝。T3 维持 Sintra 缺口 |
| E 提名作为环内显式工具 | 目标态（T4+ 演进），B 是其前置契约 |

## Consequences

- T3：geo 多样性闸为 `skeleton_only` 提供远簇成日；**不**重开提名（守 ADR-062）。
- T4：`agent-itinerary-101` / `2play-plan-102` 落地独立提名（must-see + 理由）与 chat refine；骨架以偏好消费。
- 演进：T4+ 把提名暴露为环内显式工具（E），模型按 must-see 是否为空自决调用。
- 无城市百科：geo 闸与提名 grounding 均目的地无关（ADR-042）。

## Follow-up stories

T3 骨架质量（不重开提名；ADR-042 无城市百科）拆为三故事，按序 DoD：

| Story | 范围 |
| --- | --- |
| `agent-itinerary-102` | 骨架提示：提名同款季节/偏好块（`other` 为偏好） |
| `agent-itinerary-103` | 偏好补池：`skeletonPoolQueries` locale 模板 + cap |
| `agent-itinerary-104` | geo 多样性闸：远簇独立成日（`must_include` 空仍生效） |

配套：BFF `bounds.end` = startDate+(days−1)。T4：`agent-itinerary-101` / `2play-plan-102`。

## Alternatives considered

见上表 A / B only / D / E。
