---
title: LLM 行程规划 Token 优化经验
type: ops-lesson
status: active
as_of: 2026-08-21
tags:
  - llm
  - performance
  - itinerary
related:
  - adr/ADR-032-llm-itinerary-mcp-tool-split.md
  - knowledge/agent/places-agent-loop.md
---

# LLM 行程规划 Token 优化经验

## Summary
places-agent 行程规划从纯代码切换到 LLM 后，首次耗时 1:47。通过四项 token 优化降至 ~1:00（-44%）。核心发现：LLM 耗时与 input+output token 量近似线性，减少不必要的候选信息是最高效的优化手段。

## Evidence
- 候选 15→8/type: user message 从 ~3000 token 降至 ~1500
- max_completion_tokens 4096→2048: 输出不再"想太多"
- 删除 hours/price 详情: 每条候选减少 ~20 token（LLM 自查已不依赖这些字段）
- 45s timeout: 防止极端情况卡死，自动 fallback 旧代码
- Quanzil 模型（非直连 OpenAI）本身响应较慢，~45s 是模型瓶颈

## Lesson / guidance
1. **先减 token 再考虑架构变更** — 减少候选数和描述长度是零成本高回报的优化
2. **LLM 不需要所有字段** — hours/price 等详情可以在 LLM 规划后由代码校验（Zod），不需要让 LLM 看到
3. **max_completion_tokens 应匹配实际需求** — 1 天行程 ~800 token，设 4096 让模型浪费时间"思考是否还要写更多"
4. **超时 + fallback 是必须的** — LLM 不可控，45s 超时 + 旧代码 fallback 保证用户一定能拿到结果

## Links
- [[ADR-032-llm-itinerary-mcp-tool-split]]
- [[places-agent-loop]]
