# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-18 (MVP-T8 reslice)  
**Branch:** real-agent-refactory  
**背景：** MVP-24 因质量问题暂停；插入真智能体重构（[ADR-054](./adr/ADR-054-poc-before-ui.md) / [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md)）。  
**唯一 backlog：** `[product-backlog.md](./product-backlog.md)` §0 / §1（本文件只记工作计划与下一步，不重复排期）。  
**Takeoff / after-submit：** [ADR-061](./adr/ADR-061-takeoff-11-fields-skeleton-first.md) Accepted（T2）· [ADR-062](./adr/ADR-062-mvp-t3-skeleton-vs-t4-nominate.md) **Superseded by** [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)（T3 discovery）· [ADR-063](./adr/ADR-063-skeleton-only-plan-trip.md) Accepted · [ADR-065](./adr/ADR-065-nominate-vs-skeleton-relationship.md) **Superseded by** ADR-067 · [ADR-069](./adr/ADR-069-delete-must-see-marking-and-t4-reasons.md)（T4 Cancelled）。

---

## 已完成（本批文档准备）

- [x] 删除 `agent-specs/1-agent-refactory.md`（历史 draft，内容已并入 `real-agent-refactory.md`）
- [x] `agent-specs/real-agent-refactory.md` 新增「能力清单」表（capability table）
- [x] `product-backlog.md` §0 改为 POC + MVP-T 序；§1 增 `Paused` legend，暂停 MVP-24 as-built 行，新增 POC + MVP-T1–T5 + 扩展探针行；§3 原则更新
- [x] `agent-specs/agent-stories.md` 写入 `agent-poc-01` POC GWT/AC（检查表 #1/#5/#8/#25/#28）
- [x] [ADR-050](./adr/ADR-050-where2play-no-product-llm.md) Proposed → Accepted
- [x] 新增 [ADR-054](./adr/ADR-054-poc-before-ui.md) POC 先于 UI
- [x] 新增 [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md) MVP 重切为真智能体闭环

## 已完成（POC）

- [x] `plan_trip` 默认 `runFullLoopAgent`（模型 act-or-stop）；legacy 固定管线 `PLAN_TRIP_LEGACY_FULL_LOOP=1`
- [x] intake：`geocode` → 懒建 `trip_id` → `search_places`+eligible → `commit_trip` 芯片（`must_see` + `photos[0]`）
- [x] 全环：`resolve_origin_stay` / `make_itinerary` / `plan_next_stop` / `commit_artifacts`；registry 回填（ADR-056）
- [x] `scripts/verify-poc-true-agent.ts`：fixture、不调 Google；质量检查 `no_time_overlap` / `has_afternoon` / `meal_has_card`
- [x] 验收物 `[poc-true-agent-verification.md](./poc-true-agent-verification.md)` + `[.html](./poc-true-agent-verification.html)`
- [x] `agent-poc-01` 标 Done（2026-09-07）

## 待办

### 1–2. POC / MVP-T1 / MVP-T2

- [x] POC `agent-poc-01`（见上节）
- [x] MVP-T1：`93a` / `90a` + `95`–`99` — usable Confirmed 2026-09-09
- [x] MVP-T2：Takeoff 11 → submit（`2play-plan-100` / `agent-geocode-100`）— usable Confirmed 2026-09-09 · ADR-061

### 3. MVP-T3 — Submit → assistant + skeleton（**Done** · usable Confirmed 2026-09-11）

**故事（Done）：** `2play-plan-101` · `agent-itinerary-100`  
**依赖：** MVP-T2 usable

- [x] Story 0 cleanup
- [x] 写 AC + 测试 + 实现 + DoD usable confirm（2026-09-11）

### 3b. MVP-T3+ skeleton quality（**Done** · usable Confirmed 2026-09-11）

`agent-itinerary-102`–`109` Done。配套 ADR-066 venue-type allowlist。

### 3c. MVP-T3++ LLM-driven discovery（**Done(producer)** · [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**状态：** agent `110a`–`110d` **Done(producer)** 2026-09-11；2play `103`/`104` **→ MVP-T8**。

**质量债（Done）：** `agent-quality-111` · `agent-test-112`。

### 4. MVP-T4 — Must-see + chat refine skeleton

**状态：** **Cancelled**（[ADR-069](./adr/ADR-069-delete-must-see-marking-and-t4-reasons.md)）。chat refine 并入 **MVP-T9**。

### 5. MVP-T5 — 补池 + fill + 渐进 UI（**S1–S5 Done** · TD-3–TD-7）

> **2026-09-14 合并：** 原 T5+T6 合并为一批。  
> **2026-09-18 拆分：** 剩余 TD-8/9/10 + 扩展探针 **并入 MVP-T8**。

| TD | 内容 | 状态 |
| --- | --- | --- |
| TD-3 | fill 循环过早 stop（A+B） | **Done** |
| TD-4 | HTTP answers（hotel / expand_radius） | **Done** |
| TD-5 | tokyo provider_failed | **Done** |
| TD-6 | 2play 逐站渐进（fetch filled SoT） | **Done** |
| TD-7 | 多日默认留 Day 1 | **Done** |
| TD-8 | 餐档 + directions 完整性 | **→ MVP-T8** |
| TD-9 | 硬闸复查 + deviations 终态 | **→ MVP-T8** |
| TD-10 | UI 对齐 mockup | **→ MVP-T8** |

计划真源：[`T5-plan.md`](./T5-plan.md)

### 6. MVP-T8 — 行程规划闭环（**当前 · Batch 1 of 3**）

**目标：** 完成行程规划相关全部剩余工作 + 探针/e2e 回归。

| 步 | 故事 | 范围 | 状态 |
| --- | --- | --- | --- |
| 1 | `2play-plan-103` | Assistant 骨架下 deviations 文字（i18n） | **In progress** |
| 2 | `2play-plan-104` | 扩半径 need_input 确认 UI | ToDo |
| 3 | `agent-itinerary-93b` TD-8 | 餐档 + directions 完整性 | ToDo |
| 4 | `agent-itinerary-93b` TD-9 | 硬闸复查 + ready/failed 终态 | ToDo |
| 5 | `2play-plan-90b` TD-10 | UI 对齐 mockup | ToDo |
| 6 | `agent-discover-93f` | 杭州/香港/起点卡探针 | ToDo |
| 7 | `2play-plan-90f` | 三城 + 起点卡消费 | ToDo |
| 8 | 回归 | 12-case 骨架探针 + 6-case fill 探针 + 2play e2e 套件 | ToDo |

### 7. MVP-T9 — chat 改行程（**后计划 · Batch 2 of 3**）

- `agent-chat-93e` — plan_trip 同环 chat 改行程
- `2play-plan-050` — BFF 产品 LLM 移除（ADR-050）
- `2play-plan-90e` — in-page chat 转发 agent

### 8. MVP-T10 — 出行贴士 + 保存行程（**后计划 · Batch 3 of 3**）

- `agent-tips-93d` + `2play-plan-90d` — 四卡 tips
- 保存行程闭环：`2play-plan-25` AC2–3 · `2play-saved-26` · `2play-plan-27` replan · `2play-plan-28` PDF

### 9. 暂停项复苏

- [ ] 真智能体主干稳定后，重排 §1 `Paused` 行（MVP-24 子项、profile、plan-41 等）

---

## 下一步工作

**当前下一步：MVP-T8** — 首个故事 `2play-plan-103`（deviations 文字 UI）。

| 批次 | 状态 |
| --- | --- |
| POC / T1 / T2 / T3 / T3+ | Done |
| T3++ agent | Done(producer) |
| T4 | Cancelled |
| T5 | TD-3–TD-7 Done；剩余 → T8 |
| **MVP-T8** | **In progress** |
| MVP-T9 / T10 | 后计划 |

**不在本计划：** what2eat 改动（ADR-050 D3）；2play as-built 打磨（Paused，部分 → T10）。
