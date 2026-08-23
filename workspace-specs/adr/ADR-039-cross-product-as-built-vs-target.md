# ADR-039: 跨产品能力用 as-built / target 双态标注，禁止混写「已交付」

## Status
Accepted

## Context
places-agent 与 where2play 分属独立 git 仓库与规格树，但共享同一条行程管线（L1 discover → L2 arrange → UI）。2026-08-23 对齐时发现：agent Feature **35**（Mode H `execution=host`）在 `agent-stories` 已标 **Done**，而 2play `itinerary-design` / stories / `performance.md` 仍写「Mode H 未实现」或把目标管线写成当前架构；部分文档又把「agent 已就绪」与「2play 已消费」绑成同一状态。结果是两边「决策看起来不一致」，实际是**状态粒度错误**（生产者能力 vs 消费者接线 vs 文档目标态）。

## Decision
1. **能力跨仓库时，状态至少拆两层：**  
   - **Producer（places-agent）**：HTTP/MCP 契约是否可测交付（如 Feature 35）。  
   - **Consumer（where2play / ChatBox）**：是否已按该契约接线（如 `plan-11`）。  
   任一层未完成，**不得**在另一层文档写全局「Mode H 已落地」。
2. **行程/通道专项规格必须显式分节：**  
   - **As-built（当前可跑路径）**  
   - **Target（已拍板未接线）**  
   禁止用单一架构图同时充当两者而不标注。
3. **关闭 producer Feature 的 DoD 文档门：** 更新本仓 stories/test-plan **且** 回写共享交叉引用（至少 `performance.md` § 相关行、consumer stories 的「依赖 / 阻塞」列、相关 ADR Consequences）。consumer 未接线时，阻塞表述改为「producer Done；缺口在 consumer」。
4. **「依赖 agent Feature N」仅在 N 未 Done 时成立；** N Done 后改为「消费 N」故事，不得继续写并行阻塞。

## Rationale
- 单仓库「Feature Done」无法表示跨产品交付完成；混写导致计划、验收与排障各说各话。  
- 备选「只维护 agent 文档、2play 事后追」已被本次漂移证伪。  
- 备选「文档只写目标态」会让新人把未接线路径当成生产真相（本次 itinerary-design §1）。

## Consequences
- 新增/更新跨产品能力时，按本 ADR 检查 as-built vs target 与双边状态。  
- 知识篇：[`../knowledge/web-app-development/cross-product-spec-drift.md`](../knowledge/web-app-development/cross-product-spec-drift.md)。  
- 不替代 ADR-037 产品决策；本 ADR 约束**如何记录与同步状态**。

## Date
2026-08-23
