# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-14  
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

- 固定四问 intake（hotel / start_time / must_see / other）— 前三已在起飞栏；**不再做 agent 提示必去点**（ADR-069）
- ~~必去提名与理由、聊天改骨架（→ MVP-T4）~~ — **Cancelled by ADR-069**；chat refine 并入 T8
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

### 3c. MVP 编号与近期两档（已确认）

| 编号 | 名称 | 范围 | 相对位置 |
| --- | --- | --- | --- |
| **MVP-T3++** | 智能体规划行程核心机制 | **仅** `agent-discover-110a` | **Done**（usable Confirmed 2026-09-11） |
| **MVP-T3++Q** | 骨架质量与偏差透明 | `110e`/`110b`/`110g`/`110c`/`103`/`110d`/`104` Done；`110f` Monitor | **Done** |
| ~~**MVP-T4**~~ | ~~必去提名 + 聊天 refine~~ | ~~`agent-itinerary-101` · `2play-plan-102`~~ | **Cancelled by ADR-069** |

编号约定：T3++ = 核心机制落地（发现路径替换）；T3++Q = Quality / deviations / 扩半径；~~T4~~ Cancelled by ADR-069（不再做 agent 提示必去点）。

---

### 3d. MVP-T3++ 智能体规划行程核心机制（**Done** · usable Confirmed 2026-09-11 · 仅 `110a` · [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**命名：** 智能体规划行程核心机制（Agent Planning Core Mechanism）。  
**编号：** **MVP-T3++**  
**状态：** **Done**（usable Confirmed 2026-09-11）。唯一实现单元：`agent-discover-110a`。  
**依赖：** MVP-T3 / T3+ Done。  
**设计真源：** [`agent-design.md`](./agent-specs/agent-design.md) Index → `#core-t3pp` / `#core-trip-store` / `#core-registry`。

**目标：** 用 LLM 驱动发现取代代码模板 stops-pool，建立端到端核心：智能体提名 → 清洗 → grounding → 写本 `trip.candidates` → 骨架从本 trip 池选。Registry 退化为 **cache-only**（命中跳过 search，**禁止** merge 整城库进候选池）。

#### 细切评估（为何下一批只做 110a）

| 维度 | 评估 |
| --- | --- |
| 增量交付 | `110a` 单独即可替换发现路径；`110b`–`104` 是提示/校验/UI 质量，不挡「核心机制可跑」 |
| 风险 | `110a` 是最大、最高风险切片（删模板搜 + 换发现 + cache-only）；与 deviations/扩半径捆在一起会拖长 DoD |
| 可用性 | 110a Done 后：新发现路径可用；季节仍在骨架提示（as-built）、远簇仍可能静默修补、薄池仍无扩半径确认 — 由 **T3++Q** 消化 |
| ADR-067 | todo1（方案 A 发现）+ registry cache-only + ADR-059 提名侧 → **T3++**；todo2/4b2/5/6a/6b → **T3++Q** |
| 结论 | **建议细切：下一 MVP = 仅 110a。** |

#### Scope（In）— 仅 `agent-discover-110a`

| # | 功能块 | 说明 |
| --- | --- | --- |
| 1 | **LLM OptA 发现 + grounding** | 20–30 名；无 trip_type 枚举；反模糊；季节硬规则在提名侧；起飞偏好进提名；清洗；逐名 grounding；写本 trip.candidates |
| 2 | **Registry cache-only** | 命中跳过 `searchPlaces`；miss 可选 upsert（事实 only，无 `must_see`）；**不** `mergeRegistryPlaces` |
| 3 | **ADR-059 全条件进提名 prompt** | destination / tripType / budget / startDate / days / partySize / pace / transit / startTime / origin / other |
| 4 | **Code refactor（发现路径）** | 删 `skeletonPoolQueries` / `expandPlacesForSkeleton` / enrich 侧 `mergeRegistryPlaces` |
| 5 | **骨架仍从本 trip 池选** | 资格过滤复用；骨架 LLM 排程路径保留（提示收紧属 T3++Q） |
| 6 | **测试 + 可观测** | TC-T3-110a-01～07；debug 可读 trip.candidates |

**已有、本 MVP 不重做：** Trip Store（ADR-046 Done）；起飞 11 / skeleton_only / 资格闸（T2/T3/T3+ Done）。

#### Scope（Out）— 移交 MVP-T3++Q

- 骨架提示 2a-none + other 去「偏好」→ `110b`
- 校验不修补 + `deviations` + Assistant 展示 → `110c` / `2play-plan-103`
- POI 不足扩半径 + 确认 UI → `110d` / `2play-plan-104`
- ~~用户面对必去 + 聊天 refine → MVP-T4~~ — **Cancelled by ADR-069**；**不再做 agent 提示必去点相关功能**
- fill / meals / directions / 四卡 → **T5+**
- BFF 产品 LLM 移除（`2play-plan-050`）→ gate **T5–T8**

| Story | 状态 |
| --- | --- |
| `agent-discover-110a` | **Done**（usable Confirmed 2026-09-11） |

- [x] 实现 `110a`（ATDD → TDD）+ 测试矩阵 §44（TC-T3-110a-01～07）
- [x] DoD / usable confirm（核心机制：新发现路径可出骨架）

---

### 3e. MVP-T3++Q 骨架质量与偏差透明（**Done** · ADR-067 余项）

**命名：** 骨架质量与偏差透明（Skeleton quality + transparent deviations）。  
**编号：** **MVP-T3++Q**  
**状态：** **Done**（2026-09-11；`2play-plan-104` 收尾）。`110f` 持续 Monitor，不挡交付。  
**依赖：** MVP-T3++ 核心机制落地（Done）。

#### Scope（In）

| # | 功能块 | 故事 | 状态 |
| --- | --- | --- | --- |
| 1 | 骨架提示 2a-none + other 去「偏好」 | `agent-discover-110b` | Done |
| 2 | 校验不修补 + `deviations` JSON/DB | `agent-discover-110c` | Done |
| 3 | Assistant 骨架下文字展示 deviations（i18n） | `2play-plan-103` | Done |
| 4 | POI 不足扩半径 + `need_input` | `agent-discover-110d` | Done |
| 5 | 扩半径确认 / 拒绝 UI | `2play-plan-104` | Done |
| 6 | 骨架闸门软化 + 限制条件解释字典 | `agent-discover-110e` | Done |
| 7 | 删 must_see 标记层（C） | `agent-discover-110g` | Done |

**顺序：** 已全部交付（Monitor：`110f`）。

#### Scope（Out）

- 发现路径重写（已在 T3++）
- ~~用户面对必去 / 聊天 refine → MVP-T4~~ — **T4 Cancelled by ADR-069**；chat refine 并入 T8
- fill / meals / 四卡 → **T5+**

- [x] 实现 + 测试 + DoD / usable

---

### 4. ~~MVP-T4~~ — 必去提名 + 聊天 refine 骨架（**Cancelled by ADR-069**）

**命名：** 必去提名与聊天 refine（Must-see + chat refine skeleton）。  
**编号：** ~~MVP-T4~~ — **Cancelled**。  
**状态：** **Cancelled**（2026-09-11 · [ADR-069](./adr/ADR-069-delete-must-see-marking-and-t4-reasons.md)）。  
**产品政策（2026-09-14 确认）：** **不再做 agent 提示必去点相关功能** — 不恢复 C（`must_see` 标记 / `[must-see]` 注入）、不恢复 D（必去理由 UI）、不新开任何「agent 提示必去点」故事。保留 A（110a 发现提名，无 must_see 标志）与 B（用户 `must_include` 硬闸）。骨架侧「prefer well-known attractions」是 ADR-068 节奏/重要性软偏好，**不是**必去产品功能。

**原因：** must_see 标记层（C）违背真智能体原则（代码替 LLM 预判重要性）；探针验证 110e 软偏好 + 人设可替代 C（5/5）；T4 必去理由交互与 T8 chat refine 重复。chat refine 并入 T8（`agent-chat-93e` / `2play-plan-90e`）。

| 故事 | 状态 |
| --- | --- |
| `agent-itinerary-101` | **Cancelled** |
| `2play-plan-102` | **Cancelled** |

**删除工作：** `agent-discover-110g`（删 C 代码层）归入 T3++Q，不单列 MVP。

---

### 5. MVP-T5 → T8 + 扩展探针（T3++Q 之后）

用户确认骨架后生成 `trip_details`；**逐站 / 逐日**渲染（非整日一次甩出）；以及原 backlog 能力顺延：

- [ ] **MVP-T5**：补池 + 按日 filled（无餐）+ 2play 逐日/逐站渲染（`agent-itinerary-93b` / `2play-plan-90b`）
- [ ] **MVP-T6**：餐档 + directions + 硬闸 + 2play 含餐与交通
- [ ] **MVP-T7**：四卡（artifacts）+ 2play 出行贴士页
- [ ] **MVP-T8**：chat 改行程 + 2play in-page chat（含 `2play-plan-050` ADR-050 BFF LLM 移除；含原 T4 chat refine — ADR-069）
- [ ] 扩展探针：杭州 / 香港 / 起点卡（ADR-053）

### 6. 暂停项复苏

- [ ] 真智能体主干稳定后，重排 §1 `Paused` 行（MVP-24 子项、saved、replan、export、chat resize、profile、plan-41 等）

---

## 下一步工作

**当前焦点：** **MVP-T5**（补池 + 按日 filled；T4 已 Cancelled by ADR-069）。  
**其后：** MVP-T6 → T8 + 扩展探针。

- T1 / T2 usable **Confirmed 2026-09-09**
- T3 / T3+ usable **Confirmed 2026-09-11**
- T3++（`110a`）usable **Confirmed 2026-09-11**
- **T3++Q Done 2026-09-11**（`110e`/`110b`/`110g`/`110c`/`103`/`110d`/`104`；`110f` Monitor）
- ~~T4~~ **Cancelled by ADR-069**（chat refine 并入 T8）
- **政策：** 不再做 agent 提示必去点相关功能（ADR-069；2026-09-14 再确认）
- 切分真源：ADR-061（T2）· ADR-063（`skeleton_only`）· **ADR-067**（T3++ / T3++Q）· **ADR-068**（软节奏 vs 硬安全闸）· **ADR-069**（删 must_see + T4 Cancelled）

**不在本计划：** what2eat 改动（ADR-050 D3 隔离）；2play as-built 打磨（Paused）；**agent 提示必去点**（标记/理由/必去 UI）。
