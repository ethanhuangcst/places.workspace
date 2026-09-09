# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-09  
**Branch:** real-agent-refactory  
**背景：** MVP-24 因质量问题暂停；插入真智能体重构（[ADR-054](./adr/ADR-054-poc-before-ui.md) / [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md)）。  
**唯一 backlog：** [`product-backlog.md`](./product-backlog.md) §0 / §1（本文件只记工作计划与下一步，不重复排期）。  
**Takeoff 重切草稿：** [`agent-specs/tmp-0909.md`](./agent-specs/tmp-0909.md) · [ADR-061](./adr/ADR-061-takeoff-11-fields-skeleton-first.md) Accepted（T2）。

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
- [x] 验收物 [`poc-true-agent-verification.md`](./poc-true-agent-verification.md) + [`.html`](./poc-true-agent-verification.html)
- [x] `agent-poc-01` 标 Done（2026-09-07）

## 待办

### 1. POC 实现（`agent-poc-01`，Lisbon 单城，无 UI）

- [x] 在 places-agent 实现 `plan_trip` intake + 全环（见上节）
- [x] `fetch_trip_details` 读路径（candidates / skeleton / filled）
- [x] POC 脚本：Lisbon 4 日场景；HTML + markdown 可观测
- [x] 通过判据：fixture 路径 AC 行为断言 + 质量检查；检查表 #1/#5/#8/#25/#28 无违反
- [x] DoD：人眼复核 HTML；库隔离（Prisma 测试/本地库，非生产）；用户确认 POC 可用

### 2. MVP-T1（故事 AC 已签收）

- [x] 写 AC：`agent-itinerary-93a` + `2play-plan-90a`
- [x] `agent-itinerary-93a`：4 题 need_input + 三城路由 fixture（2026-09-09）
- [x] `2play-plan-90a`：8 字段 + `/api/plan/trip` + 逐题 + session PATCH + candidates
- [x] 质量切片：`95` / `96` / `97` / `98` / `99`（`96` AC5：suggest→search，2026-09-09）
- [x] 产品闭环：Lisbon 8 项 + 4 问 + 芯片 — **usable Confirmed 2026-09-09**（ADR-060：芯片可少/空）
- [x] 收尾：[`mvp-1t-closing-plan.md`](./mvp-1t-closing-plan.md)（设计合并 + 质量门）**Confirmed 2026-09-09**

### 3. MVP-T2 — Takeoff 11 → submit（**Done · usable Confirmed 2026-09-09**）

**Scope：** 起飞栏收集 **11 个输入** 到 **提交**（含 blur 验证与可提交门禁）。**不含**提交后助手接管 / `plan_trip` 行为变更。

| 项 | 内容 |
| --- | --- |
| 故事 | `2play-plan-100`（2play）；`agent-geocode-100`（结构化 geocode） |
| 字段 | destination · tripType · budget · startDate · days · partySize · pace · transit · startTime · origin · other |
| 验证 | 目的地：geocode → `国家/地名`（海外补英文）；起点：suggest 100%/部分/不匹配页面悬浮窗；其余按控件规则 |
| 明确不做 | 助手接管、skeleton-first `plan_trip`、必去提名、去掉固定 4 问、fill |

- [x] 写 AC（`2play-plan-100` + `agent-geocode-100`）+ mock `06-plan-takeoff-11.html`
- [x] 实现 + 测试（agent geocode + where2play takeoff）
- [x] DoD / usable 确认（仅 takeoff→submit）— **Confirmed 2026-09-09**
- [x] 细节与决策：[tmp-0909.md](./agent-specs/tmp-0909.md) · [ADR-061](./adr/ADR-061-takeoff-11-fields-skeleton-first.md)

### 4. MVP-T3 — After-submit（助手 + plan_trip）

**Scope：** 提交之后 — 助手接管、`plan_trip` **只出骨架**、agent-driven `needs_input`（无固定 4 问）、必去提名带理由、聊天确认/改。

| 项 | 内容 |
| --- | --- |
| 故事 | `2play-plan-101` + agent `plan_trip` skeleton-first 故事 |
| 依赖 | MVP-T2 usable |

- [ ] 写 AC
- [ ] 实现 + 测试 + DoD / usable

### 5. MVP-T4 → T6 + 扩展探针（原 T2–T5 顺延）

原「补池 + 骨架 + 按日 filled / 餐 / 贴士 / chat」在 T3 骨架路径落地后继续，编号顺延：

- [ ] **MVP-T4**（原 T2）：补池 + 按日 filled（无餐）+ 2play 行程详情逐日渲染（`agent-itinerary-93b` / `2play-plan-90b`）
- [ ] **MVP-T5**（原 T3）：餐档 + directions + 硬闸 + 2play 含餐与交通
- [ ] **MVP-T6**（原 T4）：四卡（artifacts）+ 2play 出行贴士页
- [ ] **MVP-T7**（原 T5）：chat 改行程 + 2play in-page chat
- [ ] 扩展探针：杭州（大陆 AMAP-only / 废除 discover 扩源 / D9-D10）、香港（Google+AMAP）、起点卡（ADR-053）

### 6. 暂停项复苏

- [ ] 真智能体主干稳定后，重排 §1 `Paused` 行（MVP-24 子项、saved、replan、export、chat resize、profile、plan-41 等）

---

## 下一步工作

**下一步：开 MVP-T3（after-submit / skeleton-first `plan_trip`）。** MVP-T2 usable Confirmed 2026-09-09；不要把 T4+ 并进 T3。

- T1 usable + 收尾门 **Confirmed 2026-09-09**
- T2 takeoff→submit **usable Confirmed 2026-09-09**
- 设计草稿：[tmp-0909.md](./agent-specs/tmp-0909.md) · ADR-061 Accepted（T2）

**不在本计划：** what2eat 改动（ADR-050 D3 隔离）；2play as-built 打磨（Paused）。
