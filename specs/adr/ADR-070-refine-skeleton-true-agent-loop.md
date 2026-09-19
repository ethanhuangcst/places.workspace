# ADR-070: refine 改行程 — 骨架真智能体环（supersede ops-patch 主路径）

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Cancelled**（2026-09-18）— superseded by [ADR-071](./ADR-071-descope-in-page-refine-replan-only.md)

## Context

MVP-T9 落地了 `plan_trip` refine（[`agent-chat-93e`](../agent-specs/agent-stories.md)）：LLM 在工具环里选 `commit_trip.operations[]`（remove/replace/swap），并由 `validateRequiredDropOperations` 等硬闸校验「不去 X」是否触及正确 `stop_index`。

生产问题（上海 Day 2「第二天上午不去海洋公园…」）表明：

1. 硬闸 + BFF 在 `changed: false` 时一律返回 `play.chat.refine_no_change`，**吞掉**模型本可发出的追问。
2. ops-patch 把「理解用户意图」压成三种操作 + 索引，与 [ADR-055](./ADR-055-mvp-reslice-true-agent-loops.md) 真智能体原则冲突：应由模型判读 **改 / 追问 / 不能改**，而非代码正则闸门。
3. 规划环已是 skeleton-first + fill；refine 应 **同构**（先完整骨架 commit，再按变天 refill），而非微补丁后再猜 refill 范围。

Grounding（[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)）与无 where2play 产品 LLM（[ADR-050](./ADR-050-where2play-no-product-llm.md)）保持不变。

## Decision

### D1 — refine 主路径改为 skeleton LLM + 追问 + refill

`plan_trip` refine 环：

1. 读 Trip Store（constraints、skeleton、filled_stops、deviations、candidates、refine 线程）；**不**持久化首次规划的 hidden chat。
2. 模型 outcomes：**改**（commit 完整新骨架）/ **追问**（`stop`，`changed: false`）/ **诚实不能改**（grounding 或工具失败）。
3. `search_places` grounding 新店名；`commit_skeleton`（可扩展 `commit_trip`）写入**整天数组**，未改天原样抄回。
4. commit 成功后 **确定性** 触发现有 fill（`needs_refill`）；仅重填 skeleton attraction 名有变化的 `day_index`（diff 决定，模型可含相邻天）。
5. **不在** refine 同一轮 LLM 里逐步调 `plan_next_stop`（避免 fill 污染 refine 上下文）。

`commit_trip.operations[]` + `validateRequiredDropOperations` **降为实现细节或删除**，不再作为 refine 主路径。

### D2 — BFF 三态 reply

`POST /api/chat` 当 agent `changed: false`：

- 若模型 `reply` 为**追问或说明**，BFF **透传**模型文案；**不得**一律替换为 `play.chat.refine_no_change`。
- `play.chat.refine_no_change` 仅用于**真无操作**（用户确认不改或模型明确无改动必要）。

`changed: true` → `needs_refill: true`；**不** merge skeleton 进 filled DTO（与 hotfix 一致）。

### D3 — 不新开 where2play LLM

2play 仍只转发 agent；refine 判读在 places-agent `plan-trip-refine` 环完成（ADR-050 不变）。

### D4 — 追问边界

同一信息缺口最多追问 2 次；之后诚实停止。Prompt 禁止在槽位已够时重复问「哪一天哪个点」。

### D5 — 保留的能力

- Proximity / indoor / far_cluster 等 **语义**保留，由模型 + prompt + `search_places` 表达，而非 ops 类型枚举。
- 客户端 refine 线程、progress、session draft（`2play-refine-thread`）不变。

## Consequences

**Positive**

- 与用户心智一致：说不清就问，说清了才改。
- refine 与 T3 规划 pipeline 对齐，refill 范围由 skeleton diff 决定，可维护。
- 修复「未作改动」误报掩盖追问的 UX 问题。

**Negative / cost**

- 每次成功 refine 至少一次完整骨架 LLM + 变天 fill，比单次 replace 贵。
- 需重写 `plan-trip-refine.ts` 与 BFF reply 分支；现有 ops 单测需迁移或删。

**Migration**

1. 本 ADR Accepted → 更新 [`plan-trip-refine-t9.md`](../knowledge/agent/plan-trip-refine-t9.md)、2play/agent stories GWT、`change-log.md`（**无代码**）。
2. 下一轮实现：`plan-trip-refine` + `/api/chat` + 回归 E2E。

## References

- Knowledge: [`plan-trip-refine-t9.md`](../knowledge/agent/plan-trip-refine-t9.md)
- Stories: `agent-refine-true-agent`, `2play-refine-true-agent`
- Supersedes as primary path: ops-first refine in `agent-chat-93e` US2（保留 Done 追溯，行为由 ADR-070 覆盖）
