# 工作计划 — 真智能体重构插入

**Status:** active · as_of 2026-09-20（`2play-plan-90d` / `agent-geocode-114` **Done** usable Confirmed；**P0 = Visa track**；**P2 = 保存闭环**）  
**Branch:** real-agent-refactory · commits `5f05be0` (specs) · `6688414` (where2play) · `c2ebf05` (places-agent)  
**背景：** MVP-24 as-built 打磨暂停（[ADR-055](./adr/ADR-055-mvp-reslice-true-agent-loops.md)）；子故事见 [`product-backlog.md`](./product-backlog.md) §0.1。插入真智能体重构（[ADR-054](./adr/ADR-054-poc-before-ui.md)）。  
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

**质量债（Done）：** `agent-quality-111` · `agent-test-112` · **`agent-fill-113`**（ADR-072 fill-by-id，2026-09-19 vitest）。

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

### 6. MVP-T8 — 行程规划闭环（**Done** · usable Confirmed 2026-09-18 · Batch 1 of 3）

**目标：** 完成行程规划相关全部剩余工作 + 探针/e2e 回归。

| 步 | 故事 | 范围 | 状态 |
| --- | --- | --- | --- |
| 1 | `2play-plan-103` | Assistant 骨架下 deviations 文字（i18n） | **Done** |
| 2 | `2play-plan-104` | 扩半径 need_input 确认 UI | **Done** |
| 3 | `agent-itinerary-93b` TD-8 | 餐档 + directions 完整性 | **Done** |
| 4 | `agent-itinerary-93b` TD-9 | 硬闸复查 + ready/failed 终态 | **Done** |
| 5 | `2play-plan-90b` TD-10 | UI 对齐 mockup | **Done** |
| 6 | `agent-discover-93f` | 杭州/香港/起点卡探针 | **Done** |
| 7 | `2play-plan-90f` | 三城 + 起点卡消费 | **Done** |
| 8 | 回归 | 12-case 骨架探针 + 6-case fill 探针 + 2play e2e 套件 | **Done**（11/12 skeleton，test12 台北 known；fill HZ/Lisbon/Tokyo 100%；e2e mvp1/mvp2/mvp3/mvp10-structure/mvp-t3/mvp10-live 绿；chat02 defer MVP-T9） |

### 7. MVP-T9 — chat 改行程 — **Cancelled**（ADR-071）

改行程 = **Replan only**（确认 → 新 full-loop）。`agent-chat-93e` / `2play-plan-90e` / refine hotfix 链 **Cancelled**。`2play-plan-050`（BFF 无产品 LLM）**Done**。

**ADR-071 descope（2026-09-18）：** full-stack 删除 refine；vitest 回归绿；三仓已 push（2026-09-19）。工作区与 `origin/real-agent-refactory` 一致（本地误改已 restore）。

| 门禁 | 状态 |
| --- | --- |
| 实现 + push | **Done** |
| Vitest（descope 相关） | **Pass** |
| 浏览器 usable（takeoff → 完成 → 无 composer → Replan） | **Done** — usable Confirmed 2026-09-20 |
| DoD usable confirm | **Done**（2026-09-20）

### 8. MVP-T10 — 出行贴士 + 签证 + 保存行程

#### 8a. Done

1. `agent-tips-93d` — 四卡 **`artifacts.tips` only**（skeleton 后与 fill 并行；无 visa）← F50/F76；UI → `2play-plan-90d` · **Done**（usable Confirmed 2026-09-20）
2. `2play-plan-90d` — 出行贴士四卡 UI（消费 93d；visa 空藏 popover；**无 Orizn**）← 24-P2d · **Done**（usable Confirmed 2026-09-20）
3. `agent-geocode-114` — 目的地验真闸（城市/景点）· **Done**（usable Confirmed 2026-09-20）

#### 8b. 当前 Batch — Visa（**P0** · 一次一条）

Agent 生产面已就绪（`agent-visa-48` / `agent-trip-76` Done(producer)）。本批 **where2play-only**（+ `39` 纯 spec/mock）；**不在** 93d / **不在** 90d。

1. `2play-plan-39` — Visa slot spec（视觉真源 `06-plan.html` 卡 01；不写运行时）
2. `2play-profile-38` — Nationality field（ISO alpha-3；注册/资料 **必填**）
3. `2play-plan-94a` — 后台查询并写入 `artifacts.visa`（用户不可见）
4. `2play-plan-94b` — 按 mock 画出贴士卡 01 签证链接 + popover
5. `2play-plan-94c` — 查不到时诚实降级 · **Done**（usable Confirmed 2026-09-21）
6. `2play-plan-90e` — 贴士与 fill 同步 · **Done**（usable Confirmed 2026-09-21 · ADR-075）
7. `2play-plan-106` — 回访 hydrate `artifacts` · **Done**（usable Confirmed 2026-09-21）
8. `2play-plan-107` — 同 ISO 隐藏 / 免签展示 / 卡 01「目的地」· **Done**（usable Confirmed 2026-09-21）

#### 8c. Deferred Batch — Save（**P2**）← **当前下一步**

1. `2play-plan-25` AC2–3 — 保存含助手线程快照 + `tripId`（**Done** · usable Confirmed 2026-09-21）
2. `2play-plan-37` / 24-P1b — Saved 详情与 Plan 完成态同构 ← **当前下一步**
3. `2play-saved-26` — 已保存只读对话（依赖 1–2）
4. `2play-plan-27` — Replan 确认后新 full-loop（ADR-071 UI 已有，补闭环 AC）
5. `2play-plan-28` — Export PDF

### 9. MVP-24 用户故事（Paused 清单）

真源表：[`product-backlog.md`](./product-backlog.md) **§0.1**。此处只列开放项，不重复已 Super/Cancelled。

**已在真智能体消化或取消（勿复活 as-built 管线）：**

- 24-P0-ui-A/B/C、24-P0a（`2play-plan-37` / `41`）→ T2/T3/T8
- 24-P0b/c/d（`agent-itinerary-94` / `fill-95` / `infra-96`）**Done**
- 24-P1a hydrate → `2play-plan-105` **Done**
- 24-P4a 页内改行程、24-P5c chat 高度 → **Cancelled**（ADR-071）

**并入 T10：** tips Done；**Visa P0** — `2play-plan-39` / `2play-profile-38` / `2play-plan-94a`/`94b`/`94c`；**Save P2** — `2play-plan-25` AC2–3 / 24-P1b / `2play-saved-26` / `2play-plan-27` / `2play-plan-28`

**T10 之后仍 Paused（未排）：**

| 批次 | 故事 | 内容 |
| --- | --- | --- |
| 24-P0 | `agent-fill-67` | fill 契约 leftover |
| 24-P0 | `agent-iconic-69` | 必去地单一源（仅 tips，无 T4 理由） |
| 24-P0 | `agent-discover-97` | 2play 消费端 AMAP-only 核对 |
| 24-P1c | `agent-discover-87` | 运行时库 usable 探针收口 |
| 24-P2a | `agent-itinerary-68` | plan-nav chips CSS |
| 24-P2a | `agent-itinerary-73` | plan-46 测对齐 |
| 24-P2b | `2play-plan-37` AC15–19 余量 | place-sheet a11y |
| 24-P2c | `agent-itinerary-45` / `agent-trip-66` | arrange 硬删 consumer 收口 |

- [ ] Visa P0 完成后取 Save P2 下一条（`incremental-delivery`）
- [ ] Save P2 完成后再从本表取下一条

---

## ToDo @ 返回（2026-09-19）

> **Reminder：** 下次打开本项目时先看本节。

- [x] **P0 — 恢复 map 配额**（AMAP + Google Maps）— **Done**（2026-09-20 产品确认已恢复）
- [x] **P0 — ADR-071 usable verify：** 产品确认可用（2026-09-20）；完成态无 composer，改行程 = Replan only
- [x] **P1 — `agent-fill-113`：** ADR-072 骨架 pointer + fill 抄池 id + Google UI 名一次（2026-09-19）
- [x] **临时 1 — 助手线程对齐 mockup：** 完成态不藏旧消息；`.plan-progress` 珠+hint；骨架+fill_begin+fill **追加**；refresh 自 `GET /api/plan/current` 带 `skeleton` 恢复骨架 spine（2026-09-19）
- [x] **临时 2 — 骨架景点必须池内 native_id**（ADR-072 D2 · validate + post-attach drop · vitest TC-F114 · 2026-09-19）
- [x] **`agent-registry-115` — 可解析 native_id + 可展示 https 门**（registry/list/attach · 拒绝 verify_* / example.com · purge 脚本 · 2026-09-19）
- [x] **临时 5 — 时间线 `.slot-thumb` 空、详情有图**（AMAP http→https slim + fill `photos[0]` · ADR-051 D6 · 龙井村为 repro 非城市规则 · 2026-09-20）
- [x] **临时 6 — fill 时无LLM按规则排餐（`agent-meal-116`）** — `pickMealVenue` + Google mapper；vitest TC-M116；usable Confirmed 2026-09-20。合同 [`knowledge/agent/research_fill_rule_meals.md`](./knowledge/agent/research_fill_rule_meals.md)
- [x] **临时 3 — 搜餐超时 / Google 排餐墙钟（`agent-meal-117`）** — A+C+B；vitest TC-M117；Lisbon/台北探针。**Done** usable Confirmed 2026-09-20。
- [x] **临时 7 — Google 正餐类型+贝叶斯（`agent-meal-118`）** — **Done** usable Confirmed 2026-09-20。`primaryType`/`types[0]` 正餐闸；score m=50 C=4.0；Nearby 不覆盖 category。
- [x] **临时 4 — 临时行程草稿持久化（`2play-plan-105` / ADR-073）：** fill 写 cache；我的行程往返 hydrate；保存仍 `SavedItinerary`；下次规划/Replan 覆盖（2026-09-19）
- [x] **P1 — 开始 MVP-T10：** `agent-tips-93d` Implemented（tips-only）
- [x] **P1 — `2play-plan-90d`：** 贴士四卡 UI **Done** usable Confirmed 2026-09-20；visa 另条（现升 **Visa P0**）
- [x] **`agent-geocode-114`：** 目的地验真闸（城市/景点）**Done** usable Confirmed 2026-09-20
- [x] tips/geocode 三仓 push：`0e1a112` / `c2ebf05` / `6688414`（2026-09-20）
- [ ] **P0 — Visa track：** `39` → `38` → `94a` → `94b` → `94c`（故事确认后一次一条；当前仅 specs）
- [ ] **P2 — 保存闭环：** `25` AC2–3 → `37`/24-P1b → `26` → `27` / `28`
- [x] ADR-071 代码 + specs push；本地误改 restore（2026-09-19）

### 核实：T3 行程真源不是「整包 JSON 当账本」（2026-09-19）

**不是致命架构错误。** [ADR-046](./adr/ADR-046-trip-store-pg-memory-fetch.md) 仍成立：写工具信封不当 UI 真源；T3 `POST /api/plan/trip` 在 `skeleton_only` + `ready` 后 **必须** `POST /v1/fetch_trip_details` `{ fields: ["skeleton","constraints"] }`，从 Trip Store（PG + 热副本）读切片。实现：`where2play/app/api/plan/trip/route.ts`（先 `planTrip` 写，再 `fetchTripDetails` 覆盖 `skeleton`）。

之前说的「T3 plan_trip 是整包 JSON」只描述 **浏览器↔BFF 传输**：`authJson` 一次等 make+commit+fetch 全部结束才返回，**没有**把 `skeleton_day` / phase 流到助手。那是进度 UX 缺口，不是跳过数据库。信封里的 `data.itinerary.skeleton` 仅作 fetch 失败时的降级，不是产品真源。

**不要先做：** 恢复 T9 refine / `/api/chat`（ADR-071 Cancelled）。

---

## 下一步工作

**当前下一步：** T10 保存闭环（25/26）。Visa = **`2play-plan-94` 另条**。

| 批次 | 状态 |
| --- | --- |
| POC / T1 / T2 / T3 / T3+ | Done |
| T3++ agent | Done(producer) |
| T4 | Cancelled |
| T5 | TD-3–TD-7 Done；TD-8/9/10 → T8 Done |
| **MVP-T8** | **Done**（2026-09-18 usable Confirmed） |
| **MVP-T9 / ADR-071 descope** | **Done**（usable Confirmed 2026-09-20） |
| **`agent-fill-113`** | **Done**（2026-09-19 · ADR-072） |
| **临时队列** | **全部 Done**（含 117 / 118 usable Confirmed 2026-09-20） |
| **MVP-T10** | **当前下一步** |

**不在本计划：** what2eat 改动（ADR-050 D3）；2play as-built 打磨（Paused，部分 → T10）。
