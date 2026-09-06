---
title: 行程排程过度设计 — 三类机制叠加却仍失败 must_include
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - places-agent
  - arrange_day
  - anti-pattern
  - retrospective
  - over-engineering
related:
  - ../../adr/ADR-043-chatbox-mcp-and-cross-product-closure.md
  - ./geo-hardcode-recurrence.md
  - ./arrange-p0-nullish-slim.md
  - ./mcp-client-integration.md
---

# 行程排程过度设计 — 三类机制叠加却仍失败 must_include

## Summary

行程排程（`arrange_day`）的核心需求很简单：根据目的地与用户边界条件，排出含交通、places、restaurants 的合理多日行程。一个简单的 prompt 就能做。但为修「必去写了却没排」和「末日卡重复」两个症状，连续叠加了三类机制——P1 展示死代码、`must_include` assignment 分日表 + 确定性注入、五处城市硬编码——花了三天、烧了 $500 token，must_include 仍复现。复盘发现：**每个症状都被「加更多机制」修补，从未退一步质问机制本身**；机制总量超过功能本身，是过度设计的典型形态。

## Evidence — 三类过度设计

### 1. P1 展示死代码（宿主根本不遵守）

- `arrange-present-gate.ts` 的 `overviewEmittedSet`/`isOverviewEmitted`/`markOverviewEmitted`/`buildDayCardMarkdown`/`buildArrangePresentationBlob`/`INTERNAL_REASON_MARKERS`，加 `create-server.ts` 的 `presentation` 字段与 `already_complete` 分支。
- 目的：服务端拼好「用户可见 markdown」日卡，靠 `overview_emitted` 防末日重复展示。
- 实际：宿主（ChatBox）**完全忽略** `user_visible_markdown`/`overview_emitted`，自己拼卡。末日不重复靠的是 `host_instructions` 文本控制句（「Do NOT call arrange_day again」），不是这套展示机制。
- 信号：机制上线后症状无变化 → 该机制是死代码。

### 2. must_include assignment 分日表 + 确定性注入

- `SessionEntry.assignment: token → dayIndex` 把每个必去 token 预分配到某天；`seedMustIncludeAssignment`/`getAssignedMustIncludeTokenForDay`。
- `ensureHardMustIncludeCoverage`：LLM 没排到 focus 时，**服务端自己注入** 一个最优 place 为 `attraction`（`reason: hard_must_include`）。
- 目的：保证必去一定排进。
- 实际：注入制造**低质块**（只为满足覆盖，非 LLM 精选），且 `hard_must_include` 这个内部 reason 标记**外泄给用户**。assignment 分日表还把 day-trip 小镇抢排到无 theme 的早期日（如辛特拉被 Day 1 抢成半天）。
- 信号：为保覆盖而造的块，质量低于 LLM 自选 → 机制在跟产品目标打架。

### 3. 五处城市硬编码（见 geo-hardcode-recurrence 第三次）

- `must-see-coverage.ts` 西安簇名单 + 自动注入；`discover-dedupe.ts` 西安地标去重正则；`place-filters.ts` 地标后缀正则；`itinerary-planner.ts` 远郊行政区过滤；`discover-must-see.ts` CATALOG。
- 目的：让西安出兵马俑、过滤地标名当餐厅等。
- 实际：表面通用、实际只对西安生效，对非西安静默失效；且违反 ADR-042「源码禁止任何城市 POI 知识」原则（第三次重犯，详见 [`geo-hardcode-recurrence.md`](./geo-hardcode-recurrence.md)）。

## 为什么会过度设计（过程根因）

1. **症状驱动，逐个加机制。** 「必去没排」→ 加 assignment + 注入；「reason 外泄」→ 加 marker 跳过；「西安要出兵马俑」→ 加簇名单。每个症状都「合理」，但从未退一步看整体。
2. **不质问机制，只加不减。** 失败时第一反应是「机制还不够强」，而非「机制本身错」。assignment 没解决 → 加注入；注入造低质块 → 加 reason 标记；标记外泄 → 加跳过逻辑。层层叠加。
3. **DoD 绑锚点城而非全球能力。** 「西安绿 = must-see 完成」用表内城验收巩固坏机制，掩盖全球缺口（与硬编码重犯根因 #2 同源）。
4. **未用最简基线对照。** 没有先问「一个简单 prompt 能不能做」，直接上机制。复盘时对照才意识到核心需求一个 prompt 即可。
5. **scar tissue 累积无清理。** 每次修补加代码，旧机制不删；到复盘时三类机制叠加，总量已远超功能本身。

## Lesson / guidance

**过度设计检测信号（命中任一即退一步质问机制）：**
- 修补某症状后，加的机制**总量**已接近或超过功能本身的核心逻辑。
- 一个机制上线后，**症状无变化**（说明它是死代码，宿主不经过它）。
- 为保某指标而造的产物，**质量低于不造时**（如注入块比 LLM 自选差）。
- 同一类失败**第三次**用同类手段修（如第三次硬编码 must-see）。
- DoD 用锚点城验收，但**非锚点城从未定义**过同等能力。

**修正原则：**
- **先减后加。** 失败时先问「能否删机制」，再问「加什么」。本次 C3 精简删了三类机制后，核心反而更稳（真实搜索 + LLM 选点 + 校验 + 重试 + 硬失败 + sticky covered）。
- **保留核心，删 scar tissue。** 精简后保留：真实 POI 搜索、交通 ETA、LLM 选点 + 校验 + 重试、AbortSignal、legacy fallback、`must_include` P0 硬失败、`covered` sticky。删的是：展示死代码、assignment、确定性注入、城市硬编码。
- **用硬失败替代注入。** LLM 没排到 focus → 抛错让它重试一次，而非服务端造低质块。质量由 LLM 保证，覆盖由硬失败兜底。
- **必去知识走 LLM 推断，不进源码。** `inferMustSeeFromPool` 让 LLM 从候选池选公认必去（prompt 无城市名），对所有城市通用，替代硬编码 CATALOG。

**审稿一问（过度设计闸）：**  
「为修这个症状，我加的机制是否已接近功能本身复杂度？删掉它，靠核心 + 硬失败 + 重试，症状会不会反而更轻？」→ 会则优先删。

## Cost

- 三天、$500 token。
- must_include 在过度设计期间仍复现（assignment 抢排、注入低质、reason 外泄）。
- 精简后端到端验证通过：focus=辛特拉 covered、`inferred_must_see` 返回、用户显式 must_include 优先。

## Links

- [ADR-043](../../adr/ADR-043-chatbox-mcp-and-cross-product-closure.md) D9 精简（删 P1 死代码 + assignment 降级 + 注入改硬失败 + 删五处城市硬编码 + 必去改 LLM 推断 + 守卫测试）
- [geo-hardcode-recurrence.md](./geo-hardcode-recurrence.md)（硬编码重犯第三次，与本文「五处城市硬编码」同源）
- [arrange-p0-nullish-slim.md](./arrange-p0-nullish-slim.md)（同 arrange 路线早期 P0 教训）
- [mcp-client-integration.md](./mcp-client-integration.md)（续排措辞无法强制宿主——另一类「服务端机制管不住宿主」的过度设计边界）
