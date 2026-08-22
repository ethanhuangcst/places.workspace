# ADR-032: LLM 主导行程规划 + MCP 工具拆分

## Status
Accepted

## Context
原行程规划由 800+ 行代码硬编码（排序→插槽→计算路线），结果不像人类旅行者思维：餐食不合理、景点扎堆、节奏差、缺推荐理由。同时 MCP 客户端（ChatBox/Claude）调 plan_itinerary 需等待完整多天结果，耗时 1:47 不可接受。

## Decision
1. **LLM 主导规划：** 代码搜索候选 → 单次 LLM 调用（含自查指令）→ Zod 校验 → fallback 旧代码。不用双 LLM（Planner+Reviewer），过度设计。
2. **MCP 工具拆分：** 新增 `discover_places`（搜候选 ~5s）+ `arrange_day`（单天安排 ~10s），MCP 客户端可逐天调用。HTTP `/v1/plan_itinerary` 保留一次返回（内部 discover + arrange × N）。
3. **Token 优化：** 候选 15→8/type，max_tokens 4096→2048（arrange 单日后调至 1280），精简描述（删 hours/price），45s 超时。
4. **Feature flag：** `ITINERARY_MODE=llm|legacy`，旧代码保留为 fallback。
5. **HTTP progressive（where2play，2026-08-21 增补）：** 对 thin BFF，HTTP `discover_places` / `arrange_day` 在 `Accept: application/x-ndjson` 时推送细粒度事件——discover **每 POI 一条**；arrange **每 block/place 一条**。MCP 仍为 request/response。跨天多样性靠 `exclude_names` / 收缩候选。详见 where2play [`2play-design.md` §2.4.1](../../3.where2play/2play-specs/2play-design.md)。**Accepted mock（2026-08-21）：** where2play `06-plan*.html` 完成态 / Discover / Arrange；**行 by 行**（Highlights → place → pending → 自动切日）见 §3.5.3。
6. **LLM 硬超时（2026-08-22）：** Quanzil 下 OpenAI SDK `timeout` 不可靠（实测 hang ≫45s）；行程 LLM 以 **AbortSignal** 硬中断；仅 Zod/业务校验失败重试；超时不二次调用。where2play BFF arrange 默认 110s，abort → `errors.arrange_timeout`。

## Rationale
- **单 LLM vs 双 LLM：** 双 LLM 延迟翻倍（~20s→~40s），成本翻倍，质量提升有限（prompt 自查已覆盖大部分审查项）。
- **MCP 拆分 vs 单工具：** MCP 协议是 request-response，不支持 streaming。拆成 discover+arrange 让 MCP 客户端天然支持逐天调用（每步 ~10s），无需 SSE。
- **HTTP NDJSON：** where2play 需要同态实时与逐 place 渲染；在 HTTP 通道补齐细粒度推送，不强迫 MCP 改协议。
- **Token 优化 vs 换模型：** 减 token 是最低成本手段（不依赖外部模型变更），从 1:47 降到 ~1:00。
- **AbortSignal vs SDK timeout：** SDK 超时对自定义 baseURL（Quanzil）未硬停；AbortSignal 与地图适配器一致，避免 BFF 先掐、agent 仍挂。

## Consequences
- 行程质量依赖 LLM prompt 质量 — 需持续优化 itinerary-planner.md
- Quanzil 模型仍然偏慢（~45s/call）— 未来可考虑更快模型或按天分批并行
- what2eat 零改动 — plan_itinerary 接口不变
- 旧代码保留增加代码量 — 稳定后（MVP-7+）可清理
- 超时失败对用户可见为 `arrange_timeout`，不再误报为笼统 `provider_failed`

## Date
2026-08-21（硬超时增补 2026-08-22）
