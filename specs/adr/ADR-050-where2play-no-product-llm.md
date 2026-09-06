# ADR-050: where2play 零产品 LLM（初排 / 助手走 places-agent）

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Proposed**（2026-09-05）— 目标架构；as-built 在实现切片落地前仍可能持产品 `QWEN_*` / `OPENAI_*`。  
规范细节：[`../agent-specs/real-agent-refactory.md`](../agent-specs/real-agent-refactory.md)。  
细化检查表：[`knowledge/agent/real-agent-refinement-checklist.md`](../knowledge/agent/real-agent-refinement-checklist.md)。

## Context

[ADR-036](./ADR-036-where2play-assistant-quanzil.md) / [ADR-037](./ADR-037-where2play-plan-l2-quanzil.md) 将 where2play 行程助手与 Plan L2 放在 **产品侧 LLM**（后由 [ADR-047](./ADR-047-qwen-primary-llm.md) 统一为 Qwen）。后果：

1. **两个规划脑** — agent 与 2play 各持模型；prompt / as-built 易漂移（ADR-039）。
2. 助手可编造第 6 题必去店名，与 agent 验真池不一致。
3. 产品方向改为 places-agent **真智能体环**：对外 `plan_trip` + `fetch_trip_details`；问卷、芯片、四卡、排程、改行程一律在 agent。

需要明确：**where2play 不再持产品 LLM**。

## Decision

### D1 — where2play 零产品 LLM

1. where2play **不得**为排程、行程助手、贴士散文、改行程持有或调用 `QWEN_*` / `OPENAI_*`。
2. 初排 / 收边界 / 必去芯片 / 四卡 / 填站 / 自然语言改行程 → places-agent（`plan_trip`；读 `fetch_trip_details`）。
3. where2play BFF：渲染 agent `need_input`、回传答案、调 HTTP、hydrate fetch 切片。无工具选择、无本地模型补全。

### D2 — 对既有 ADR 的关系（落地时再改状态）

| ADR | 本 Proposed 下的目标 |
| --- | --- |
| ADR-036 | 实现切片完成后 **Superseded**：助手不再默认产品 LLM |
| ADR-037 | 实现切片完成后 **Superseded**：L2 不再在 2play BFF |
| ADR-047 D1「三个可部署体…where2play BFF」 | **修订**：where2play **排除**；主 LLM 仍在 places-agent（及 what2eat 若保留产品路径） |

Accepted 正文本轮 **不整篇改写**；以本 ADR + `real-agent-refactory.md` 为 target 真源。

### D3 — what2eat 不在范围

what2eat 是否继续持产品 LLM **另议**；本 ADR **不改** what2eat 产品密钥、Decide、页内 chat。

what2eat 对 agent 的 HTTP 面保持：`geocode`、`search_restaurants`、`get_place_details`、`chat`。禁止将这些重指向 `plan_trip`，禁止因本 ADR 关闭 **places-agent** 上的 Qwen（2eat chat 工具环仍走 agent LLM）。行程工具与 Trip 账本对 2eat 不可见、不可强制。细则见 [`real-agent-refactory.md`](../agent-specs/real-agent-refactory.md)「what2eat 隔离」。

### D4 — MCP 宿主不变

ChatBox / Cursor 仍经 MCP 调 places-agent；纪律风险见既有 knowledge。与「2play 零 LLM」正交。

## Consequences

- **正：** 单脑排程；芯片与骨架同一验真池；密钥与 prompt 集中在 agent。
- **负：** 2play 去掉产品 key 与本地 arrange/chat 路径；需实现切片与文档同窗（ADR-039）。
- **中性：** as-built 在切片完成前可能仍出现产品 env；文档须双态标注（target vs as-built）。

## References

- [real-agent-refactory.md](../agent-specs/real-agent-refactory.md)
- [ADR-036](./ADR-036-where2play-assistant-quanzil.md)、[ADR-037](./ADR-037-where2play-plan-l2-quanzil.md)、[ADR-047](./ADR-047-qwen-primary-llm.md)
- [ADR-046](./ADR-046-trip-store-pg-memory-fetch.md) fetch-only
- [ADR-001](./ADR-001-thin-app-agent-split.md)

## Date

2026-09-05
