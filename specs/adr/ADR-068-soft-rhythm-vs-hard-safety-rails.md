# ADR-068: 软节奏 vs 硬安全闸（Soft rhythm vs hard safety rails）

**Status:** Accepted  
**Date:** 2026-09-11  
**Story:** `agent-discover-110e`（Done · usable Confirmed 2026-09-11）  
**Context:** MVP-T3++Q · skeleton quality  
**Supersedes:** 无（细化 ADR-067 的校验不修补原则）  
**Superseded by:** 无  
**Related:** [ADR-067](ADR-067-llm-driven-discovery-replaces-stops-pool.md)（discovery）· [ADR-066](ADR-066-venue-type-allowlist-vs-city-poi.md)（venue-type allowlist）· [ADR-042](ADR-042-no-city-encyclopedia-in-source.md)（no city encyclopedia）· [ADR-069](ADR-069-delete-must-see-marking-and-t4-reasons.md)（删 must_see，本 ADR 为前提）

---

## 背景

上海亲子 3 天把「上海迪士尼乐园」排成约 1/4 天。根因：

- 骨架把 `pace` 当**数字配额**（medium ≤ 5），强制每天 ≥ 2 个景点（`minAttr` 硬下限）。单景点 AM/PM 拆分被 min-2 闸挡住，乐园无法占全天。
- 填站 `attractionDwellMinutes` 无乐园/度假区档位，统一 45–60min，与全天场馆不符。
- `pace` 被当成程序硬配额，而非「动线节奏语义」交给 LLM 判断。

这是真智能体原则的反例：代码替 LLM 预判密度与节奏，挡住了模型对「乐园该占一天」的合理判断。

## 决策

把骨架闸门分成两层：**软节奏**（交给 LLM）与**硬安全闸**（代码强制）。

### 1. Pace = 软信号，非硬上限

- 删除 `minAttr` 硬下限（不再强制每天 ≥ N 景点）。
- `trimPaceOverages` 只裁**极端**超额（景点数 > `paceStopLimit` + `PACE_SOFT_MARGIN`=2）。接近上限的日（如 medium 上限 5 排 6）留给 LLM 判断。
- 提示把 pace 描述为节奏区间（tight ~5–6、medium ~3–5、relaxed ~2–3）+ 乐园/远日游例外，而非硬数字。

### 2. 天数 = 硬安全闸

- `validateSkeleton` 在 `skeleton.days.length !== numDays` 时硬拒。
- `ensureFarClustersOwnDays` 不拆出超过 numDays 的天（远景点留在主日内）。
- 省略 numDays 时保持向后兼容（不做计数检查）。

### 3. 午餐规则软化，不静默改写

- `reseatLateLunchStops` 不在单景点日把午餐强制挪到唯一景点之前。
- `splitSingleAttractionDays` 可保留 AM/全天同一场馆。

### 4. 乐园/度假区 = 全天停留档

- `attractionDwellMinutes` 对 venue-type 标记（乐园/度假区/主题公园 / theme park / resort / amusement）返回 ≥ 180min，而非通用 45/60min。
- 标记为**目的地无关的 venue-type 词汇**（ADR-066），非城市 POI 表（ADR-042）。

### 5. 11 项限制条件解释字典注入骨架提示

- `buildConstraintGlossary` 逐条说明在场条件（destination / tripType / budget / startDate / days / partySize / pace / transit / startTime / origin / other）对行程边界的影响。
- pace 描述为节奏语义（区间 + 乐园/远日游例外），i18n key-based（CN/HK/TW/EN）。
- 景点名仍只来自 grounded 候选池。

## 保留的硬安全闸（仍硬拒）

- 跨日复用同一景点（same `native_id` 或 name）
- 城市名当站（destination city as stop）
- 裸区域别名（whole area as stop）
- schema 缺失 / JSON 无效
- `must_include` 未覆盖
- 天数不匹配（本 ADR 新增）

## 后果

正面：

- 密度/节奏交给模型判断；代码只守结构完整与安全。符合真智能体原则（trust the model, get out of the way）。
- 单景点全天（乐园）不再被凑数填充；上海迪士尼可占一天。
- 为 ADR-069 删除 must_see 标记层（C）铺路：110e 软偏好「prioritize well-known attractions」+ 资深规划专家人设已可替代 C（探针 5/5 验证：兵马俑 3/3、迪士尼全日、Belém 一日游均不降）。

负面 / 风险：

- LLM 偶尔可能排过密或过松；由 `trimPaceOverages` 兜底极端超额，其余靠提示与软偏好引导。
- 软化后单测需明确区分「软偏差」（记录到 deviations）与「硬拒」（重试/失败），不能混用同一断言。

## 实现

- `places-agent/src/core/make-itinerary.ts`：`paceStopLimit` + `PACE_SOFT_MARGIN`；`validateSkeleton` 加 `numDays` 硬闸；`buildSkeletonUserMessage` 注入节奏语义与乐园例外。
- `places-agent/src/core/places-ontology.ts`：`buildConstraintGlossary` 11 项 i18n 字典；`attractionDwellMinutes` 乐园/度假区档。
- `places-agent/prompts/overlays/itinerary-skeleton.md`：pace 软规则、乐园全天、self-check 第 7 条。
- 测试：`make-itinerary.test.ts`、`eligible-attraction.test.ts`、`geo-bounds.test.ts`、`itinerary-skeleton-overlay.test.ts`。
