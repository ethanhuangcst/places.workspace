# ADR-037: where2play 初排 L2 亦由本应用 OPENAI_CN 执行

> 旧称 Quanzil；现称 OPENAI_CN（[ADR-041](./ADR-041-openai-cn-replaces-quanzil.md)）。文件名保留。

## Status
Accepted（修订 [ADR-036](./ADR-036-where2play-assistant-quanzil.md) 决策第 2、3 条中「初排/replan 仍经 agent arrange」的部分）。**关闭包 / as-built 真源：** [ADR-043](./ADR-043-chatbox-mcp-and-cross-product-closure.md)（2play = Mode H + enrich；ChatBox MCP = 强制 agent）。

## Context
ADR-036 将 **行程助手** 定为 where2play BFF → 本应用 OPENAI_CN，但 **生成行程 / 重新规划** 仍走 places-agent `discover_places` + `arrange_day`。实测主路径瓶颈即 agent 侧非流式结构化 `arrange_day`（常 ~40–45s/日，易超时 502），导致 Plan 页无法验收助手等下游能力。产品要求：**最核心的生成行程页面也转到新方案**（与助手同一产品 OPENAI_CN、流式通道），地图与候选发现仍集中在 agent。

## Decision
1. **L1 Discover（地图候选）** 仍经 places-agent：`POST /v1/discover_places`（及配图 join 所需原始候选）。2play **不含** `AMAP_*` / `GOOGLE_MAPS_*`。
2. **L2 Schedule（按天形成 hour-by-hour）** 改由 **where2play BFF 本应用 OPENAI_CN** 执行：`POST /api/plan` / `/api/plan/replan` 在 discover 之后，对 dayIndex=1…N 调用产品 `OPENAI_*`（gpt-5.4），**不再**默认调用 agent `arrange_day` / `plan_itinerary`。
3. **流式 UX 不变：** BFF 仍向 UI 发 NDJSON（`candidate_place` → `place` / `day_done` / `done`）；目标为 **真流式或按 block 尽早推送**（优先首日首站可见），不得退回「整日 JSON 完成后才首次出站」。
4. **行程助手（ADR-036）** 保持 BFF OPENAI_CN；小改用 `itineraryPatch`。整单重做走 `/api/plan/replan`（本 ADR 的 L1+L2 管线），**禁止**助手默认 `plan_itinerary`。
5. **可选降级（显式）：** 仅当产品开关允许且 OPENAI_CN 失败时，可回退 agent `arrange_day`；默认关闭。不得用代码启发式骨架冒充主路径排程（performance.md v2）。
6. **一套排程 prompt（Mode H）：** 初排与 MCP 共用 agent `buildSchedulePrompt`。**MVP-3 目标：** 每日 `POST /v1/arrange_day` `execution=host` 取 `system_prompt` / `user_prompt`，再由 2play OPENAI_CN 执行。**禁止** execution=agent 作为 2play 主路径默认。

## Rationale
- 同模型下，把 L2 从「agent 工具内等满 JSON」挪到 **2play 可流式的 BFF 通道**，对齐 ChatBox 体感与助手方案。
- L1 留在 agent：vendor 密钥与 QLP 搜词策略不复制进 2play。
- 修订 ADR-036「初排仍 agent」：助手 alone 无法解除 Plan 页主路径阻塞。

## Consequences
- 更新 [`2play-design.md`](../../3.where2play/2play-specs/2play-design.md) §2.1 / §2.4.1 / §2.4.4 / §2.8；[`2play-prod-specs.md`](../../3.where2play/2play-specs/2play-prod-specs.md)；[`2play-stories.md`](../../3.where2play/2play-specs/2play-stories.md) / test-plan；[`performance.md`](../../1.places-agent/agent-specs/performance.md)；[`2.architecture.md`](../2.architecture.md)。
- 实现：`plan-day-by-day`（或后继模块）L2 改调 OPENAI_CN；契约测 mock LLM；live DoD 仍要求真实 discover（vendor）+ 真实/批准沙箱 OPENAI_CN。
- places-agent `arrange_day` 仍服务 MCP / 其他 HTTP 调用方（含 Mode H）；**不再是 2play 主初排的 LLM 执行器**。
- protect-eng：2play 已需 `OPENAI_*`（助手）；初排共用同一套密钥。

### As-built vs follow-on（2026-08-23）

| 层 | As-built | Follow-on |
| --- | --- | --- |
| L1 | agent `discover_places` | 不变 |
| L2 prompt | 2play 本地 `buildArrangeDayMessages` | **MVP-3 / `plan-11`：** agent `execution=host` |
| L2 LLM | 2play OPENAI_CN | 不变 |
| Transit / 时长 | 估时合成（已知 ~15min 问题） | **MVP-3 / `plan-13`：** `legs_to_here` |
| 地标 | 不稳定 | **MVP-3：** host prompt + ADR-038 + E2E 断言 |

## Date
2026-08-22（修订注记 2026-08-23）
