# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-06  
**Branch:** real-agent-refactory  
**背景：** MVP-24 因质量问题暂停；插入真智能体重构（[ADR-054](./adr/ADR-054-poc-before-ui.md) / [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md)）。  
**唯一 backlog：** [`product-backlog.md`](./product-backlog.md) §0 / §1（本文件只记工作计划与下一步，不重复排期）。

---

## 已完成（本批文档准备）

- [x] 删除 `agent-specs/1-agent-refactory.md`（历史 draft，内容已并入 `real-agent-refactory.md`）
- [x] `agent-specs/real-agent-refactory.md` 新增「能力清单」表（capability table）
- [x] `product-backlog.md` §0 改为 POC + MVP-T 序；§1 增 `Paused` legend，暂停 MVP-24 as-built 行，新增 POC + MVP-T1–T5 + 扩展探针行；§3 原则更新
- [x] `agent-specs/agent-stories.md` 写入 `agent-poc-01` POC GWT/AC（检查表 #1/#5/#8/#25/#28）
- [x] [ADR-050](./adr/ADR-050-where2play-no-product-llm.md) Proposed → Accepted
- [x] 新增 [ADR-054](./adr/ADR-054-poc-before-ui.md) POC 先于 UI
- [x] 新增 [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md) MVP 重切为真智能体闭环

## 待办

### 1. POC 实现（`agent-poc-01`，Lisbon 单城，无 UI）

- [ ] 在 places-agent 实现 `plan_trip` intake 路径：`geocode` 锚点 → 懒建 `trip_id` → LLM 提名必去 → `search_places`+eligible → `commit_trip` 写 `candidates`（`must_see` + `photos[0]`）
- [ ] `fetch_trip_details(fields:["candidates"])` 读路径
- [ ] POC 脚本 / CLI：跑 Lisbon 单城，输出 trip JSON + 芯片渲染
- [ ] 通过判据：AC1–AC4 全绿 + 检查表 #1/#5/#8/#25/#28 无违反
- [ ] DoD：可观测物可人眼复核；行为断言；库隔离

### 2. MVP-T1（POC 通过后）

- [ ] `agent-itinerary-93a`：`plan_trip` intake + `need_input` 问题返回
- [ ] `2play-plan-90a`：5 题问卷渲染 + 第 6 题芯片勾选回传（无产品 LLM）
- [ ] 写 AC（POC 通过后）
- [ ] 闭环验收：Lisbon 5 题 + 芯片 usable

### 3. MVP-T2 → T5 + 扩展探针

- [ ] MVP-T2 补池 + 骨架 + 按日 filled（无餐）+ 2play 逐日渲染
- [ ] MVP-T3 餐档 + directions + 硬闸 + 2play 含餐与交通
- [ ] MVP-T4 四卡（artifacts）+ 2play 出行贴士页
- [ ] MVP-T5 chat 改行程 + 2play in-page chat
- [ ] 扩展探针：杭州（大陆 AMAP-only / 废除 discover 扩源 / D9-D10）、香港（Google+AMAP）、起点卡（ADR-053）

### 4. 暂停项复苏

- [ ] 真智能体主干稳定后，重排 §1 `Paused` 行（MVP-24 子项、saved、replan、export、chat resize、profile、plan-41 等）

---

## 下一步工作

**立即开始：** POC `agent-poc-01` 实现（待办 §1）。一次一条故事到 DoD（`incremental-delivery`）。

- 代码仓：places-agent（`plan_trip` / `fetch_trip_details` / 内部工具）
- 验收：检查表 #1/#5/#8/#25/#28；AC 见 [`agent-specs/agent-stories.md`](./agent-specs/agent-stories.md)
- POC 通过前不写 MVP-T1+ 的 AC，不接 2play UI

**不在本计划：** what2eat 改动（ADR-050 D3 隔离）；2play as-built 打磨（Paused）。
