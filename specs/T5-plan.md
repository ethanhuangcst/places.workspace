# MVP-T5 开发计划 — 补池 + 按日 filled（含餐档）+ directions + 硬闸 + 2play 逐日/逐站渐进渲染

**Status:** planning · as_of 2026-09-14
**批次：** MVP-T5（合并原 T5 + 原 T6，2026-09-14 决定）
**前置：** MVP-T3++（agent-discover-110a→110d / 2play-plan-103/104）usable Confirmed
**故事：** agent-itinerary-93b（agent）· 2play-plan-90b（2play）— 拆分见 §6
**UI 真源：** 2play-specs/ui-mockup/06-plan-skeleton.html · 06-plan-fill-timeline.html
**设计真源：** 2play-specs/2play-design.md §4.11（渐进渲染已补全）· agent-specs/agent-design.md §4 填细节
**相关 ADR：** ADR-050（BFF 零产品 LLM）· ADR-054/055（真智能体闭环）· ADR-063（skeleton-first）· ADR-068（软节奏 vs 硬安全闸）· ADR-069（删 must_see 标记）
**探针脚本：** places-agent/scripts/probe-t5-fill-review.ts（回归用）

> **本文件是计划，不是实现。** 遵守 `incremental_delivery`：一次一故事到 DoD。开发顺序见 §3。

---

## 1. 合并理由（原 T5 + 原 T6）

| 维度 | 拆两批（原计划） | 合并一批（本计划） |
| --- | --- | --- |
| 返工 | T5 无餐填景点 → T6 加餐推移时钟 → T5 时间线失效 | 餐与景点同次填，时钟一次定型 |
| 实现复用 | 需拆 plan_next_stop 一体步骤 | 复用 MVP-10（agent-fill-44 Done）现有一体实现 |
| DoD | 逐日 filled 无餐半成品 | 逐日 filled 含餐 + 交通接近真智能体 Target |

plan_next_stop（places-agent/src/core/plan-next-stop.ts）现有实现单次调用即完成：景点卡片 + 邻站搜餐 + directions ETA + 时钟累加。拆开等于改实现且引入时钟重排返工。

---

## 2. 探针发现（2026-09-14，clean HEAD = MVP-T3++Q）

用 prompt-test-case.md 6 组跑 plan_trip(skeleton_only=false) 全环（探针脚本 probe-t5-fill-review.ts）：

| 探针 | 状态 | 填充完整度 | 关键观察 |
| --- | --- | --- | --- |
| test1 上海亲子 3 天 | ready | 3/15 (20%) | 仅 D1 填到 12:30；D1 下午+晚餐、D2/D3 全空 |
| test2 杭州情侣 3 天 | ready | 3/18 (17%) | 仅 D1 填到 10:47；无餐；D2/D3 全空 |
| test3 西安历史 3 天 | needs_input | — | 卡 intake 问 hotel（无起点） |
| test4 里斯本情侣 3 天 | ready | 3/15 (20%) | 仅 D1 填到 12:30；D2/D3 全空 |
| test5 东京动漫 1 人 | failed | — | errors.provider_failed（502） |
| test6 江阴 14 天 | needs_input | — | 卡 intake 问 hotel（无起点） |

**工具调用模式（3 个 ready 一致）：** make_itinerary → plan_next_stop ×3 → commit_artifacts → stop。模型填 3 站就主动 stop，剩余 80%+ 未填却标 status: ready。

**根因：** fill 循环控制——模型过早调 stop。单站填充机制本身可用（已填 3 站结构正确：时间 + transit + 坐标 + 卡片）。这是 T5 的 **blocker**。

**附带发现：**
- HEAD 已有 `answers.expand_radius`；hotel/origin `needs_input` 仍无法经 HTTP 答 → S2
- tokyo provider_failed → 独立供应商问题，单独排查（S3）
- 脏工作区若剥掉 T3++Q 扩半径/answers/day-count，S1 前须恢复 HEAD

---

## 3. 修正后的开发顺序（遵守 incremental_delivery）

| 步 | 内容 | 产出 |
| --- | --- | --- |
| 1 | 确认技术方案（fill 循环 + 决策表；TD-2 延后） | TD-0 A+B、TD-1 已锁定；TD-2a 软锁、TD-2b 探针后再评 |
| 2 | 修 fill 循环 + 探针验证（紧耦合小循环：改 → 探针 → 迭代，直到 6 探针 fill ≥80%） | S1 / TD-3 到 DoD |
| 3 | 更新计划 + 拆功能清单（T5 → 单故事） | §6 故事表 |
| 4 | 增量交付（specs → tests → code，一故事到 DoD 再开下一） | S2…S8 逐个 |

**顺序修订（2026-09-14）：** 先 **TD-3（S1）**，再根据探针评估 **TD-2b（当日 review LLM）**。TD-2a（逐站 fill 无新 LLM）对 S1 **软锁定**，不挡开工。

**day-review LLM：** 须完整日才有 review 对象；归 **TD-2b / TD-11**，不进 S1。

---

## 4. 步骤 1 — 技术方案（先与用户确认）

**基线：** as-built = places-agent **HEAD（MVP-T3++Q）**。若工作区脏树删掉了 `shouldAskExpandRadius` / `answers` / day-count deviations，S1 前必须先恢复 HEAD。  
**TD-0（2026-09-14）：** fill 控制锁定 **A+B**（见 §4.1）。探针 ≤3 轮仍 fill 低于 80% 时再议是否降级 C，不自动切 C。  
**TD-1（2026-09-14）：** §4.2 / §4.3 已锁定。  
**TD-2 拆分（2026-09-14）：** 见 §4.4 — **TD-2a 软锁**（S1 不引入 fill LLM）；**TD-2b**（当日 review）延后到 TD-3 探针后。

### 4.1 fill 循环控制策略 — **已锁定 A+B**

探针证实模型填 3 站就 stop。选项对照（C 未选，仅作降级后备）：

| 选项 | 做法 | 优点 | 风险 |
| --- | --- | --- | --- |
| A 改 stop 工具描述 | stop 描述加「仅当所有日所有非 stay 站已 plan_next_stop 后才可调」 | 最小改动；保留模型自主 | 模型可能仍误判 |
| B 改系统提示 | 系统提示显式「必须按日逐站 plan_next_stop 至 trip_complete，再 commit_artifacts + stop」 | 明确指令 | 同 A，依赖模型服从 |
| C 代码硬循环 | 代码驱动 plan_next_stop 循环至 cursor 到 trip_complete，模型不决定 stop | 确定性 100% | 偏离真智能体；与 ADR-068 软闸有张力 |

**锁定（用户 2026-09-14）：A + B。** S1 改 `stop` 工具描述 + `fullSystemPrompt`，循环仍由模型驱动。若探针 ≤3 轮仍 fill 低于 80%，停下来与用户确认是否降级 C（不自动切）。降级 C 若发生，再落 ADR。

### 4.2 决策表 T1–T6（agent）— **已锁定**

| # | 决策（锁定） | 状态 / 说明 |
| --- | --- | --- |
| T1 | 复用现有 `plan-next-stop.ts`（`planNextStop` / `planNextStopFill`）：单次调用 = 景点 + 餐档 + directions/启发式 + 时钟 | 锁定。MVP-10 / Feature 44 Done。不拆餐到后批。 |
| T2 | 薄池扩半径：`shouldAskExpandRadius` + `need_input` `expand_radius`（HEAD 已有 HTTP `answers.expand_radius`） | 锁定（HEAD）。脏树回归须先恢复。hotel/origin `needs_input` 属 **S2**（expand_radius 不覆盖）。 |
| T3 | directions 失败 → fill 路径启发式 leg（`source: "heuristic"` / `transit_outcome` heuristic\|partial）。**不**在 `plan_next_stop` 声称 `errors.directions_unavailable`（该 key 仅 timed 路径） | 锁定。S6 可选用 UI/API 暴露启发式标记；不必发明 timed 路径的 error key。 |
| T4 | 硬闸沿用 ADR-068：day count === `numDays`、跨日唯一、pool-only、`must_include` 覆盖 | 锁定（HEAD `validateSkeleton`）。节奏仍为软闸。 |
| T5 | 餐档失败 F91：禁 `meal_skipped`；走廊 800m→2km→5km；仍无则保留空餐槽名 | 锁定（`meal-corridor.ts` + fill 路径）。 |
| T6 | **目标（S7）：** `ready` 仅当全日填完 + 硬闸过；`failed` 当硬闸不过（附 `deviations`） | 锁定为 **目标，非 as-built**。现状：`filledStops.length > 0` 即 `ready`（故探针早停仍标 ready）。S7 验收改语义。 |

### 4.3 决策表 U1–U5（2play）— **已锁定**

| # | 决策（锁定） | 状态 / 说明 |
| --- | --- | --- |
| U1 | 渐进事件传输继续用 NDJSON（`authNdjsonEvents`） | 锁定。MVP-3 `2play-plan-32`。 |
| U2 | 逐站渐进：每次 `plan_next_stop` 后 **BFF** `fetch_trip_details({ fields: ["filled","cursor"] })` 为 SoT → 发 `stop_filled`（映射 slot）→ 客户端追加 `liveSlots`。客户端**不**自行再拉当日切片 | 锁定。对齐 §4.11 / fetch-only；关闭当前信任 `fill.display` 信封的缺口。工作在 **S4**。 |
| U3 | 默认 Day tab 留 Day 1（`is-on`）；后续日 `queued` / filling；**禁止** `focusDayIndex` 跟随填充日自动跳转 | 锁定为目标。规格已写；代码仍自动切日（`plan-page.tsx`）。**S5** 改行为。 |
| U4 | 未填站：`skeleton-day` + `skeleton-stop.is-pending` 与已填 slot 共存 | 锁定。焦点日已有；未完成非焦点日浏览属 S4/S5 UX。 |
| U5 | 助手 route-spine 复用 `plan-fill-route.tsx` | 锁定。24-P0-ui-B。 |

### 4.4 LLM Prompt 方案（TD-2 拆分）

| 步骤 | 是否 LLM | 说明 |
| --- | --- | --- |
| 骨架生成 | 是（已有） | T3++ 已验 |
| 补池 query | 否（T3++ OptA 提名已 LLM；T5 复用池） | 不新写 |
| 逐站 fill | 否（无新 LLM） | `plan_next_stop` 纯代码 + directions |
| 餐档现搜 | 否 | search_places(restaurant) 代码 |
| 硬闸复查 | 否 | 代码校验 |
| 全环编排提示 | 仅 A+B 微调 | `stop` 描述 + `fullSystemPrompt`（S1）；不算「填充用 LLM」 |
| 当日 review LLM | **待决（TD-2b）** | 须 TD-3 探针后再评；不进 S1 |

| ID | 决策 | 状态 |
| --- | --- | --- |
| **TD-2a** | 逐站 fill / 餐档 / 硬闸 **不**引入新 LLM；S1 只动 A+B 编排提示 | **软锁定**（2026-09-14）— 默认执行，不挡 TD-3 |
| **TD-2b** | 代码 fill 完成后是否用 LLM 做 **当日 review** | **延后** — TD-3 探针达标后再评（与 TD-11 合并评估） |

**S1 scope 硬边界：** 不改 `plan_next_stop` 为 LLM；不实现 day-review。
---

## 5. 步骤 2 — 修 fill 循环 + 探针验证

**目标：** 6 探针 fill 完整度 ≥80%（xian/jiangyin 需先补 HTTP answers 通路才能跑通；见 S2）。

**小循环：**
1. 按 §4.1 **A+B** 改 `stop` 工具描述 + `fullSystemPrompt`（不改代码硬循环）
2. 跑 probe-t5-fill-review.ts（6 探针）
3. 看填充完整度 + 每日 metrics（站数 / 末时 / transit / 回弹 / 餐）
4. 不达标 → 调整 → 重跑（≤3 轮）
5. 达标 → S1 故事 DoD（探针证据 + 单测）

**验收：** ready 案例每日均有午餐+晚餐，末时落在 pace 窗，fill ≥80%；needs_input 案例能经 HTTP answers 推进到骨架/fill。

### 5.1 探针轮次 1（A+B 落地后，2026-09-14）

| 探针 | 状态 | 填充完整度 | 观察 |
| --- | --- | --- | --- |
| shanghai | ready | **15/15 (100%)** | 全日有 meals=2；末时 19:00；回弹偏多 |
| hangzhou | ready | **18/18 (100%)** | 同上 |
| lisbon | ready | **18/18 (100%)** | 同上；fill_t≈86s |
| xian / jiangyin / tokyo | 未本轮必跑 | — | 仍属 S2/S3；S1 验收用 ready 子集 |

**结论：** A+B **首轮达标**（ready 子集 fill 100% ≥80%）。过早 stop 已解除。质量问题（回弹、餐 slot 标记、末时齐 19:00）留给 TD-2b / S6 评估，不挡 S1 fill 循环关闭。

**代码：** `FULL_LOOP_STOP_TOOL_DESCRIPTION` + `buildFullLoopSystemPrompt`（`plan-trip.ts`）；单测 `MVP-T5 S1 A+B` 绿。
---

## 6. 步骤 3 — 更新计划 + 拆功能清单

T5 拆为单故事，按依赖排序。每故事独立到 DoD 再开下一。

| # | 故事 | 范围 | 依赖 | 验收 |
| --- | --- | --- | --- | --- |
| S1 | 修 fill 循环过早 stop | A+B：`stop` 描述 + `fullSystemPrompt`；探针 ≥80% | TD-0 已锁定 | **Done**（ready 子集 100%） |
| S2 | HTTP answers 通路 | dispatch 传 answers 给 need_input（hotel/expand_radius） | S1 | xian/jiangyin 经 HTTP 推进到骨架 |
| S3 | tokyo provider_failed 排查 | 供应商 502 单独查 | 独立 | tokyo 不再 provider_failed |
| S4 | 逐站渐进渲染（2play） | BFF `fetch_trip_details(filled)` 为 SoT → NDJSON `stop_filled` → 客户端追加 `liveSlots`；`skeleton-stop.is-pending` 共存 | S1 | 浏览器逐站追加可见；slot 不来自 `plan_next_stop` 信封 |
| S5 | 多日默认留 Day 1（2play） | Day tab 默认 Day 1；后续日 queued；禁止 focusDayIndex 自动跟填 | S4 | 多日案例停在 Day 1 |
| S6 | 餐档 + directions 完整性 | 每日午餐+晚餐；directions ETA；失败 → 启发式 leg（可暴露 heuristic 标记，不用 `errors.directions_unavailable`） | S1 | 探针每日有餐；directions 失败有启发式降级 |
| S7 | 硬闸复查 + deviations 终态 | day count + 跨日唯一 + pool-only + must_include；`ready` 仅全日填完+硬闸过；failed 附 deviations | S6 | 硬闸过 ready；不过 failed+deviations |
| S8 | UI 对齐 mockup | 06-skeleton / 06-fill-timeline 逐元素对齐 | S4 S5 | 浏览器视觉对齐 mockup |

**注：** S1 是 blocker，必须先 Done。S2/S3 可与 S1 后并行（不同故事）。S4–S8 依赖 S1。

---

## 7. 步骤 4 — 增量交付（specs → tests → code）

每故事执行顺序：
1. 更新 specs（agent-design / 2play-design / prompt-test-case 如需）
2. 写/改 tests（红）
3. 改 code（绿）
4. 跑探针 + 单测 + 浏览器验证
5. DoD checklist + retrospective
6. 用户 usable confirm 后开下一故事

**day-review LLM（TD-2b / TD-11）：** 不占 S1。TD-3 探针后评估：若 fill ≥80% 仍有节奏/回弹/餐问题，再论证 agent 侧日级 review（新故事 + 可能 ADR）；若代码 fill 质量已够则可不做。

---

## 8. 风险

| 风险 | 缓解 |
| --- | --- |
| A+B 仍不达标 | 停下来与用户确认是否降级 C；若切 C 再落 ADR |
| tokyo provider_failed 阻塞 S1 探针 | S1 验收用 5 探针（排除 tokyo），S3 独立修 |
| HTTP answers 改动牵动 intake 流 | S2 独立故事，先写测试再改 dispatch |
| 渐进渲染与 skeleton 共存布局错乱 | S4 先写 2play-design 渲染节，再改组件 |

---

## 9. ToDo 汇总

| ID | 内容 | 步骤 | 状态 |
| --- | --- | --- | --- |
| TD-0 | 锁定 §4.1 fill 控制策略为 A+B（探针不达标再议 C） | 1 | **confirmed**（2026-09-14） |
| TD-1 | 锁定 §4.2/4.3 决策表 T1–T6 / U1–U5 | 1 | **confirmed**（2026-09-14） |
| TD-2a | 逐站 fill 无新 LLM（S1 仅 A+B） | 1 | **soft-locked**（2026-09-14） |
| TD-2b | 探针后评估「代码 fill + 当日 review LLM」 | 探针后 | pending（与 TD-11 合并） |
| TD-3 | S1 修 fill 循环 + 探针 ≥80% | 2 | **Done**（2026-09-14；探针 100% + 单测；agent-only usable via probe） |
| TD-4 | S2 HTTP answers 通路 | 3 | pending |
| TD-5 | S3 tokyo provider_failed | 3 | pending |
| TD-6 | S4 逐站渐进渲染 | 4 | pending |
| TD-7 | S5 多日默认留 Day 1 | 4 | pending |
| TD-8 | S6 餐档 + directions 完整性 | 4 | pending |
| TD-9 | S7 硬闸复查 + deviations | 4 | pending |
| TD-10 | S8 UI 对齐 mockup | 4 | pending |
| TD-11 | 与 TD-2b 合并：评估 day-review LLM | 后 | pending |

**下一步：** S1 / TD-3 **Done**。评 **TD-2b**（当日 review LLM）或开 **S2**（HTTP answers）。
