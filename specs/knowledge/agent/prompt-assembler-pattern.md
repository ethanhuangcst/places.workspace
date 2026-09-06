---
title: Prompt Assembler 设计模式
type: design-direction
status: active
as_of: 2026-08-21
tags:
  - prompt-engineering
  - architecture
  - i18n
related:
  - adr/ADR-032-llm-itinerary-mcp-tool-split.md
  - knowledge/agent/llm-itinerary-token-optimization.md
---

# Prompt Assembler 设计模式

## Summary
places-agent 采用"基础 + 片段拼接"的 prompt 组装模式，而非场景模板或模板引擎。这是三种方案中最简单的，适合多场景（搜餐厅、搜景点、行程规划、闲聊）+ 多语言（EN/CN/HK/TW）的组合。

## Evidence
- 评估了三种方案：(1) 基础+片段拼接 (2) 场景模板 (3) 混合 overlay
- 场景模板会导致角色定义重复 N 遍（N = 场景数 × 语言数）
- 混合 overlay 需要管理两级文件（base + overlay），复杂度偏高
- 基础+片段最简单：一个 base 不重复，片段纯追加，新场景 = 新 .md 文件

## Pattern

```
prompts/
  base.en.md          # 角色 + 通用规则（英文）
  base.zh.md          # 角色 + 通用规则（中文）
  overlays/
    meal-search.md     # 餐厅搜索追加片段
    place-search.md    # 景点搜索追加片段
    itinerary-planner.md  # 行程规划追加片段（含 JSON schema + 自查指令）

assembleSystemPrompt({ locale, intent, budget, glossary })
  → base.{locale}.md + overlays/{intent}.md + inline budget/time-of-day
```

## Lesson / guidance
1. **Simplicity 原则** — 用文件系统组织 prompt，不引入模板引擎
2. **片段是纯追加** — 不需要条件分支、变量替换，只是字符串拼接
3. **新增场景零改动** — 新场景 = 新 .md 文件 + assembleSystemPrompt 加一个 case
4. **Budget/time-of-day 内联** — 不值得单独成文件的小片段，直接在代码中拼接

## Links
- [[ADR-032-llm-itinerary-mcp-tool-split]]
- [[llm-itinerary-token-optimization]]
