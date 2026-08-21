# ADR-032: LLM 主导行程规划 + MCP 工具拆分

## Status
Accepted

## Context
原行程规划由 800+ 行代码硬编码（排序→插槽→计算路线），结果不像人类旅行者思维：餐食不合理、景点扎堆、节奏差、缺推荐理由。同时 MCP 客户端（ChatBox/Claude）调 plan_itinerary 需等待完整多天结果，耗时 1:47 不可接受。

## Decision
1. **LLM 主导规划：** 代码搜索候选 → 单次 LLM 调用（含自查指令）→ Zod 校验 → fallback 旧代码。不用双 LLM（Planner+Reviewer），过度设计。
2. **MCP 工具拆分：** 新增 `discover_places`（搜候选 ~5s）+ `arrange_day`（单天安排 ~10s），MCP 客户端可逐天调用。HTTP `/v1/plan_itinerary` 保留一次返回（内部 discover + arrange × N）。
3. **Token 优化：** 候选 15→8/type，max_tokens 4096→2048，精简描述（删 hours/price），45s 超时。
4. **Feature flag：** `ITINERARY_MODE=llm|legacy`，旧代码保留为 fallback。

## Rationale
- **单 LLM vs 双 LLM：** 双 LLM 延迟翻倍（~20s→~40s），成本翻倍，质量提升有限（prompt 自查已覆盖大部分审查项）。
- **MCP 拆分 vs 单工具：** MCP 协议是 request-response，不支持 streaming。拆成 discover+arrange 让 MCP 客户端天然支持逐天调用（每步 ~10s），无需 SSE。
- **Token 优化 vs 换模型：** 减 token 是最低成本手段（不依赖外部模型变更），从 1:47 降到 ~1:00。

## Consequences
- 行程质量依赖 LLM prompt 质量 — 需持续优化 itinerary-planner.md
- Quanzil 模型仍然偏慢（~45s/call）— 未来可考虑更快模型或按天分批并行
- what2eat 零改动 — plan_itinerary 接口不变
- 旧代码保留增加代码量 — 稳定后（MVP-7+）可清理

## Date
2026-08-21
