# ADR-074: 若骨架绑餐，仅变体 A（fill 仍算路）

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Proposed**（2026-09-20）— 评估结论，**未实施、未取代 ADR-049 D3**。立项修订餐档规则时必须先 Accepted 本 ADR（或后继）。

**Related:** [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) D3 · [ADR-063](./ADR-063-skeleton-only-plan-trip.md) · [ADR-022](./ADR-022-timed-itinerary.md) · [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md) · [ADR-050](./ADR-050-where2play-no-product-llm.md)

**Does not supersede ADR-049** until a follow-up story implements variant A.

## Context

产品评估过「取消骨架只定景点/餐档，改为 LLM 在骨架阶段定起点+景点+餐厅并落 `native_id`，fill 只按 id 拉详情」。真智能体原则要求：模型做判断，**时长与商家 id 须经工具验真**。Transit 切法有三种，取舍会改 T3 首屏与事实闸。

证据与墙钟：[skeleton-meals-vs-fill-eval.md](../knowledge/agent/skeleton-meals-vs-fill-eval.md)。

## Decision

### D1 — 否决变体 B

禁止「骨架写出 transit 时长/模式、fill 只 `get_place_details`、不再 Directions」。模型或启发式编 duration 抵触 ADR-022 与填细节事实闸。

### D2 — 变体 C 须显式废止 ADR-063

「骨架环内做完搜餐 + Directions + Details 再 commit」符合真智能体持环，但 **首屏 = 填完**，与 ADR-063 `skeleton_only` 先出骨架冲突。若选 C，须 **Accepted 修订/废止 ADR-063**，不得 silently 拖长 T3。

### D3 — 若改 ADR-049 D3，只允许变体 A

- 骨架 LLM **只从 grounded PlaceCard 抄** stay / 景点 / 餐的 `(provider, native_id)`（同 ADR-067，禁止编造 id）。
- Fill **仍 Directions**（及缺图时 Details）；可去掉走廊 `searchRestaurants`。
- 餐进骨架须校验 id ∈ 池、跨日店不重复；接受 make 重试面回升（ADR-049 原 502 根因）。
- 禁止城市餐厅百科（ADR-042）。质量用 destination-agnostic 闸（评分、拒内部/食堂类名），不是杭帮菜表。

### D4 — 现状保持直到有实施故事

未 Accepted 实施前：**继续 ADR-049 D3**（骨架餐档 + fill 现搜）。本 ADR 只锁 **将来切法**，不改代码。

## Rationale

| 切法 | 真智能体 | T3 首屏 | 总墙钟 |
| --- | --- | --- | --- |
| A | 判断在模型，路在供应商 | 骨架变慢（搜餐前移） | 接近现行；杭州餐已秒级，台北 Google 搜餐会恶化首屏 |
| B | 否 | 假加速 | 省 Directions，腿不可信 |
| C | 是 | 毁掉 skeleton-first | ≈ 现行填完 |

真正吃时间的是 Directions + Google `searchText` 乘法，不是「fill 少一轮选店 LLM」。

## Consequences

- 立项故事须引用 D1–D3；台北路径仍应先做临时 3（Nearby / 搜次封顶），否则 A 把超时搬到 T3。
- 不自动改 `make_itinerary` / `plan_next_stop`。

## Date

2026-09-20
