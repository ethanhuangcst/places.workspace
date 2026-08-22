---
title: LLM 行程规划 Token 优化经验
type: ops-lesson
status: active
as_of: 2026-08-22
tags:
  - llm
  - performance
  - itinerary
  - quanzil
related:
  - adr/ADR-032-llm-itinerary-mcp-tool-split.md
  - knowledge/agent/places-agent-loop.md
  - knowledge/llm/quanzil-gateway.md
---

# LLM 行程规划 Token 优化经验

## Summary
places-agent 行程规划从纯代码切换到 LLM 后，首次耗时 1:47。通过四项 token 优化降至 ~1:00（-44%）。核心发现：LLM 耗时与 input+output token 量近似线性，减少不必要的候选信息是最高效的优化手段。

**2026-08-22（西安 arrange）：** discover 正常；`arrange_day` 可挂到 **180s** 客户端 abort。OpenAI SDK `{ timeout: 45_000 }` 对 Quanzil **不硬停**；`catch` 对任意错误重试会把墙钟翻倍。改为 **AbortSignal 硬中断**；仅 Zod/校验失败重试；超时立即失败。arrange：`temperature 0.35`、`max_completion_tokens 1280`；BFF arrange 默认 **110s**，abort → `errors.arrange_timeout`（非笼统 `provider_failed`）。Live：~45s 返回 502，不再 180s 挂死。

## Evidence
- 候选 15→8/type: user message 从 ~3000 token 降至 ~1500（prompt 已截断；discover 池可更大）
- max_completion_tokens 4096→2048（多日）；arrange 单日再降至 **1280**
- 删除 hours/price 详情: 每条候选减少 ~20 token
- SDK timeout 不可靠；AbortSignal ~45s 硬停（ADR-032 §6）
- Quanzil 本身偏慢；超时失败 ≠ 进程崩溃

## Lesson / guidance
1. **先减 token 再考虑架构变更** — 减少候选数和描述长度是零成本高回报的优化
2. **LLM 不需要所有字段** — hours/price 等详情可以在 LLM 规划后由代码校验（Zod）
3. **max_completion_tokens 应匹配实际需求** — 单日 ≪ 多日；勿为单日留 4096
4. **硬超时 ≠ SDK timeout** — Quanzil 自定义 baseURL 必须以 AbortSignal（或等价）硬中断
5. **超时不重试 LLM** — 仅校验失败带错误上下文重试一次；避免 45s×2
6. **BFF 超时与 agent 对齐** — arrange ≥ 一次硬超时 + 一次校验重试 + buffer；错误 key 区分 timeout

## Links
- [[ADR-032-llm-itinerary-mcp-tool-split]]
- [[places-agent-loop]]
- [[quanzil-gateway]]
