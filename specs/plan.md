# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-11  
**Branch:** real-agent-refactory  
**背景：** MVP-24 因质量问题暂停；插入真智能体重构（[ADR-054](./adr/ADR-054-poc-before-ui.md) / [ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md)）。  
**唯一 backlog：** [`product-backlog.md`](./product-backlog.md) §0 / §1（本文件只记工作计划与下一步，不重复排期）。  
**Takeoff / after-submit：** [ADR-061](./adr/ADR-061-takeoff-11-fields-skeleton-first.md) Accepted（T2）· [ADR-062](./adr/ADR-062-mvp-t3-skeleton-vs-t4-nominate.md) **Superseded by** [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)（T3 discovery）· [ADR-063](./adr/ADR-063-skeleton-only-plan-trip.md) Accepted · [ADR-065](./adr/ADR-065-nominate-vs-skeleton-relationship.md) **Superseded by** ADR-067。

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

### 1–2. POC / MVP-T1 / MVP-T2

- [x] POC `agent-poc-01`（见上节）
- [x] MVP-T1：`93a` / `90a` + `95`–`99` — usable Confirmed 2026-09-09
- [x] MVP-T2：Takeoff 11 → submit（`2play-plan-100` / `agent-geocode-100`）— usable Confirmed 2026-09-09 · ADR-061

### 3. MVP-T3 — Submit → assistant + skeleton（**Done** · usable Confirmed 2026-09-11）

**ADR：** [ADR-062](./adr/ADR-062-mvp-t3-skeleton-vs-t4-nominate.md)（切片边界；发现路径见 ADR-067）· [ADR-063](./adr/ADR-063-skeleton-only-plan-trip.md)  
**故事（Done）：** `2play-plan-101` · `agent-itinerary-100`  
**依赖：** MVP-T2 usable

**Scope（In）：**

| # | 项 |
| --- | --- |
| 0 | **Cleanup：** **Done** — `mvp-1t-closing-plan` + chat dumps → [`archive/`](./archive/); `tmp-0909` stub |
| 1 | 起飞提交后 **助手接管**（隐藏 takeoff；打开 `plan-nav`） |
| 2 | BFF 调 agent **`plan_trip`** → 持久 **`trip_id`** |
| 3 | 助手窗口 **进度文案**（phase → i18n；非 2play 侧 LLM 旁白） |
| 4 | 按起飞 11 项生成 **行程骨架**（make/commit skeleton only） |
| 5 | 主区用 **现行 as-built UI** 展示骨架（constraints + day tabs + skeleton slots） |
| 6 | **复用 `/debug/plan`** 展示 plan 信息与 **stops-pool** |

**Scope（Out — 明确不做）：**

- 固定四问 intake（hotel / start_time / must_see / other）— 前三已在起飞栏；must-see 延到 T4
- 必去提名与理由、聊天改骨架（→ **MVP-T4**，且依赖 T3++）
- `plan_next_stop` fill / meals / directions / 贴士四卡全量（→ T5+）

- [x] Story 0 cleanup
- [x] 写 AC（`2play-plan-101` + `agent-itinerary-100`）+ 测试矩阵（`2play-test-plan` §13 · `agent-test-plan` §40）
- [x] 实现 + 测试（agent TC-T3-100 · 2play TC-T3-101 · `make test-e2e-mvp-t3`）
- [x] DoD usable confirm（2026-09-11）

### 3b. MVP-T3+ skeleton quality（**Done** · usable Confirmed 2026-09-11）

| Story | 范围 | 状态 |
| --- | --- | --- |
| `agent-itinerary-102` | 骨架提示：季节 + 结构化起飞偏好 | Done |
| `agent-itinerary-103` | 偏好补池 queries（无城市百科） | Done |
| `agent-itinerary-104` | geo 远簇独立成日 | Done |
| `agent-itinerary-105` | 资格闸：乐园 allow + resort | Done |
| `agent-itinerary-106` | 亲子 query：主题公园 | Done |
| `agent-itinerary-107` | 亲子提示排序 + overlay 去品牌 | Done |
| `agent-itinerary-108` | 资格闸泄漏：停车点/充电站/公交站 | Done |
| `agent-itinerary-109` | 起飞 11 项完整进提示（日期/budget/软偏好） | Done |

配套：BFF `bounds.end` = start+(days−1)。ADR-066 venue-type allowlist。

**As-built 债（移交 T3++ / ADR-067）：** 模板池 `skeletonPoolQueries` / 静默修补 `ensureFarClustersOwnDays` 为过渡；季节软规则、远簇天数漂移、小目的地 POI 薄 → 由 `agent-discover-110*` 消化，不挡 T3 Done。

### 3c. MVP-T3++ LLM-driven discovery（**下一实现切片** · [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**状态：** AC stubs Ready · **未开实现**。首个实现单元：`agent-discover-110a`。

| Story | 范围 | 状态 |
| --- | --- | --- |
| `agent-discover-110a` | OptA 提名 → 清洗 → grounding → candidates/registry；删模板搜 | AC Ready |
| `agent-discover-110b` | 骨架提示 2a-none + other 去「偏好」 | AC Ready |
| `agent-discover-110c` | 校验不修补 + `deviations` JSON/DB | AC Ready |
| `2play-plan-103` | Assistant 骨架下文字展示 deviations（i18n） | AC Ready |
| `agent-discover-110d` | POI 不足扩半径 + need_input | AC Ready |
| `2play-plan-104` | 扩半径用户确认 UI | AC Ready |

**配套决定（已记入 ADR-067）：** OptA 提示；trip_type 无枚举；other 去偏好标签；校验不修补；扩半径需用户确认。

### 4. MVP-T4 — Must-see + chat refine skeleton

**Scope：** 推荐必去点并给出理由；通过助手聊天收集更多输入并 refined 骨架。  
**故事（待开）：** `2play-plan-102` · `agent-itinerary-101`（暂定编号）  
**依赖：** **MVP-T3++ 落地**（发现路径已换；T4 复用同一 nominate 能力做用户面对 must-see）

- [ ] 写 AC
- [ ] 实现 + 测试 + DoD / usable

### 5. MVP-T5 → T8 + 扩展探针

用户确认骨架后生成 `trip_details`；**逐站 / 逐日**渲染（非整日一次甩出）；以及原 backlog 能力顺延：

- [ ] **MVP-T5**（原 T4）：补池 + 按日 filled（无餐）+ 2play 逐日/逐站渲染（`agent-itinerary-93b` / `2play-plan-90b`）
- [ ] **MVP-T6**（原 T5）：餐档 + directions + 硬闸 + 2play 含餐与交通
- [ ] **MVP-T7**（原 T6）：四卡（artifacts）+ 2play 出行贴士页
- [ ] **MVP-T8**（原 T7）：chat 改行程 + 2play in-page chat
- [ ] 扩展探针：杭州 / 香港 / 起点卡（ADR-053）

### 6. 暂停项复苏

- [ ] 真智能体主干稳定后，重排 §1 `Paused` 行（MVP-24 子项、saved、replan、export、chat resize、profile、plan-41 等）

---

## 下一步工作

**MVP-T3 / T3+ Done**（usable Confirmed 2026-09-11）。下一步开 **MVP-T3++**（`agent-discover-110a` 起），**不要**直接开 T4。不要把 fill/meals 并进 T3++。

- T1 / T2 usable **Confirmed 2026-09-09**
- T3 / T3+ usable **Confirmed 2026-09-11**
- 切分真源：ADR-061（T2）· ADR-063（`skeleton_only`）· **ADR-067**（T3++ discovery；supersedes ADR-062/065 discovery 关系）

**不在本计划：** what2eat 改动（ADR-050 D3 隔离）；2play as-built 打磨（Paused）。
