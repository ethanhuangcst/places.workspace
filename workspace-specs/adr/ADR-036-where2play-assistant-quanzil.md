# ADR-036: where2play 行程助手使用本应用 OPENAI_CN（方案 B）

> 旧称 Quanzil；现称 OPENAI_CN（[ADR-041](./ADR-041-openai-cn-replaces-quanzil.md)）。文件名保留。

## Status
Accepted — **部分修订**：决策第 2–3 条中「初排 / replan 仍经 agent `arrange_day`」已由 [ADR-037](./ADR-037-where2play-plan-l2-quanzil.md) 取代。助手路径（本 ADR 决策 1、4、5）仍有效。

## Context
行程助手（Plan 页内嵌 Chat）原定稿：where2play **不含** `OPENAI_*`，`POST /api/chat` 转发 places-agent `POST /v1/chat`（与 what2eat 一致）。实测与性能分析表明：同为 OPENAI_CN（gpt-5.4）时，ChatBox 在**对话通道流式写**墙钟与体感均明显优于 agent 工具环内非流式结构化补全；助手若经 `/v1/chat` 且可能调用 `plan_itinerary`，仍有数十秒级风险。产品已设计「行程助手」UI，需要可接受的改行程延迟。备选：**A** 继续经 places-agent chat；**B** where2play BFF 直连 OPENAI_CN 跑助手对话（地图/初排仍走 agent）。

## Decision
1. **行程助手（`POST /api/chat`）采用方案 B：** where2play BFF **持有**产品侧 OPENAI_CN 配置（`OPENAI_*` 或等价），对流式调用模型；**浏览器仍不直连** LLM（密钥只在服务端）。
2. **初排 / 重新规划** 仍经 places-agent：`discover_places` + `arrange_day`（及门面 `plan_itinerary` 若使用），**不**因助手改道而把地图密钥放进 2play。
3. **助手职责边界：** 在**已有** `ItineraryDto` 上做自然语言小改（回复流式上屏 + `itineraryPatch` 或完整替换 DTO 更新中部时间轴）。**禁止**助手路径默认触发整单 `plan_itinerary`；整单重做走 `POST /api/plan/replan` → agent。
4. **需要地图能力时：** BFF 仍 HTTP 调 places-agent `search_*` / `geocode` / `navigate`（可选），再把结果注入本轮助手上下文；不把 AMAP/Google key 下沉到 2play。
5. 本决策 **修订** 此前「where2play 永不持有 OPENAI」的表述；what2eat 是否跟进 **不在本 ADR 范围**（默认仍经 agent，除非另开 ADR）。

## Rationale
- **B vs A：** 同模型下，助手生成放在 2play 对话/SSE 通道，对齐 ChatBox「流式、任务轻、早结束」；避免 `/v1/chat` 工具环误触重规划。
- **仍用 agent 做初排/地图：** 保持 ADR-008 行程引擎与 vendor 密钥集中；B 只放开**产品对话改行程**这一层。
- **密钥不进浏览器：** 满足原安全表「浏览器永不持有 LLM key」。

## Consequences
- 须更新 [`2play-design.md`](../../3.where2play/2play-specs/2play-design.md) §2.1 / §2.4.3 / §2.8；实现需新增 BFF→OPENAI_CN 流式与 patch 契约测试。
- where2play 部署增加 OPENAI_CN 相关 env（名称见设计 §2.8）；**protect-eng**：写入 `.env*` 前须用户确认。
- places-agent `/v1/chat` 对 2play 助手不再是主路径（其它调用方仍可用）。
- 与 [`performance.md`](../../1.places-agent/agent-specs/performance.md) v2「2play 可接 OPENAI_CN」对齐为：**助手 = 本应用 LLM**。
- **后续：** 初排 L2 亦迁至本应用 OPENAI_CN — 见 [ADR-037](./ADR-037-where2play-plan-l2-quanzil.md)。

## Date
2026-08-22
