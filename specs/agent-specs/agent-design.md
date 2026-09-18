# places-agent — 技术设计

**places-agent** 可部署单元的设计文档：工具核心（HTTP + MCP）与运营者管理后台合为**一个进程**。像素细节：见下文第 §12 节。验收标准：[`agent-stories.md`](./agent-stories.md)。测试方案：[`agent-test-plan.md`](./agent-test-plan.md)。家族技术栈：[`../3.tech-specs.md`](../3.tech-specs.md)。信任模型：[`../2.architecture.md`](../2.architecture.md)。

这是一份实现级设计文档。技术栈版本与供应商接口端点见 `3.tech-specs.md`。

**状态：** 草稿 — 每次只实现一个用户故事。

## Index

| 节 | 内容 |
| --- | --- |
| [智能体规划行程核心机制](#core-t3pp) | **下一 MVP（T3++）** · 数据流图 · 时序图 · 能力合同 |
| [Trip Store](#core-trip-store) | PG 权威 + 内存热副本 · `trip_id` / `fetch_trip_details` |
| [Registry](#core-registry) | cache-only · 不补池 · `(provider, native_id)` |
| [MVP-T3 skeleton-first](#core-t3) | **Done** · as-built 时序图 · phase 合同 |
| [第三方工具调用](#core-mcp-hosts) | **Cursor / ChatBox** · MCP 传输 · 配置 · 时序图 · 宿主契约 |
| [真智能体 — 能力与原则](#core-principles) | 能力清单 · 对外方法 · 内部意图 · 提示组合 |
| [§1 目标与非目标](#1-目标与非目标) | 运行时 · 模块 · 工具核心 · 适配器 · HTTP/MCP · 数据 · 认证 · 管理后台 |
| [§19 Visa](#19-orizn-签证-adapter--visa_requirement-工具mvp-112026-09-01-已实现) | Orizn REST adapter |

**排障 / 新故事真源：** 上表前五节（T3++ → Trip Store → Registry → T3 → 第三方 MCP）+「真智能体」。历史 as-built 见 [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md)。

### Target / as-built — 真智能体行程编排（2026-09-11）

| 项 | 合同 | 说明 |
| --- | --- | --- |
| where2play LLM | **零产品 LLM** | [ADR-050](../adr/ADR-050-where2play-no-product-llm.md) **Accepted** |
| 对外编排 | `plan_trip` + `fetch_trip_details` | 「真智能体」+ T3 / T3++ |
| 下一 MVP | **MVP-T5 fill + 渐进 UI** | [`T5-plan.md`](../T5-plan.md) · TD-8 next |
| Registry | **cache-only** | 不 merge 整城库进 trip 池（ADR-056 amendment） |
| Trip Store | PG + 内存热副本 | [ADR-046](../adr/ADR-046-trip-store-pg-memory-fetch.md) |
| T3 skeleton | **Done**（usable 2026-09-11） | as-built 仍为模板池；质量债 → T3++ |
| MCP 宿主 | Cursor `/mcp` · ChatBox `/sse` | 见 [第三方工具调用](#core-mcp-hosts) |
| T4 | must-see + chat refine | **Cancelled**（ADR-069） |
| T5+ | fill / 四卡 / chat | **In progress**（T5 TD-8 next） |

**排障与新故事以 Index 前列（T3++ / Trip Store / Registry / T3）+「真智能体」为准。** 历史 as-built 见 [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md)。Paused 流见 [`product-backlog.md`](../product-backlog.md)。

---

<a id="core-t3pp"></a>

## MVP-T3++ 智能体规划行程核心机制（**下一 MVP 关键功能** · [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**命名：** 智能体规划行程核心机制（Agent Planning Core Mechanism）。  
**目标：** 用 LLM 驱动发现取代代码模板 stops-pool，建立端到端核心机制：智能体提名 → 清洗 → grounding → 写入本 `trip.candidates` → 骨架从本 trip.candidates 选。Registry 退化为 **cache-only** grounding 加速器（不补池、不 merge 整城库）。  
**状态：** **Done(producer)** 2026-09-11（`110a`–`110d`）；2play 消费端 `103`/`104` AC Ready。当前主线 → **MVP-T5**（[`T5-plan.md`](../T5-plan.md)）。  
**关联 ADR：** [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)（主）、[ADR-056](../adr/ADR-056-registry-backfill-semantics.md)（cache-only amendment）、[ADR-042](../adr/ADR-042-no-city-encyclopedia-in-source.md)、[ADR-050](../adr/ADR-050-where2play-no-product-llm.md)、[ADR-059](../adr/ADR-059-non-null-conditions-not-dropped.md)、[ADR-066](../adr/ADR-066-venue-type-allowlist-vs-city-poi.md)。

### 1. 设计原则

| 原则 | 说明 |
| --- | --- |
| 真智能体 | 代码提供能力（geocode、searchPlaces grounding、validation）；模型提供判断（发现、排程、deviation 理由） |
| 无城市百科 | 发现不靠 `CATALOG` / 城市→POI 表；LLM 提名是目的地无关机制（ADR-042） |
| Registry cache-only | Registry 仅按 `(provider, native_id)` 缓存已 grounding 的事实卡；命中跳过 `searchPlaces`；**禁止** `mergeRegistryPlaces` 把整城库并入 trip 候选池 |
| 校验不修补 | `ensureFarClustersOwnDays` 等从「静默修补」改为「校验只读」；漂移写 `deviations` 透明展示，不回改 LLM 输出 |
| 透明 deviation | 远簇漂移、POI 不足等以 `deviations` 文本在骨架下展示，不开警告面板 |
| i18n | 所有用户可见字符串走 key（`deviations.*`、`phase.*`、`need_input.*`） |

### 2. 数据流图（end-to-end）

```mermaid
flowchart LR
  subgraph Input
    TK[Takeoff11 边界]
  end
  subgraph Discovery[发现环 - LLM 驱动]
    NOM[LLM OptA 提名<br/>20-30 名<br/>行程类型→venue 类别]
    CLN[清洗<br/>去重/规范化]
    GRD[逐名 grounding<br/>searchPlaces / registry 命中跳过]
    CAN[写入 trip.candidates<br/>本 trip only]
  end
  subgraph Skeleton[骨架环]
    SK[LLM 排骨架<br/>池=本 trip.candidates]
    VAL[校验不修补<br/>deviations 可选]
    COM[commit skeleton]
  end
  subgraph Cache[Registry - cache-only]
    REG[(POI registry<br/>provider+native_id)]
  end
  subgraph Vendors
    MAPS[AMAP / Google<br/>searchPlaces / geocode]
  end

  TK --> NOM
  NOM --> CLN --> GRD
  GRD -- miss --> MAPS
  MAPS --> GRD
  GRD -- hit --> REG
  GRD -- miss+upsert --> REG
  GRD --> CAN
  CAN --> SK --> VAL --> COM
  REG -. 命中跳过 .-> GRD
```

**关键不变量：** `trip.candidates` 只来自 LLM 提名 + grounding 的本 trip 名单；registry 永不作为发现源注入候选池。

### 3. 时序图 — 发现 + 骨架（T3++ Target）

```mermaid
sequenceDiagram
  autonumber
  participant BFF as where2play BFF
  participant PT as plan_trip tool
  participant Loop as Agent tool loop (Qwen)
  participant Reg as POI registry (cache-only)
  participant Maps as AMAP / Google
  participant Trip as Trip Store
  participant Fetch as fetch_trip_details

  BFF->>PT: plan_trip(Takeoff11, skeleton_only=true, locale)
  PT->>Trip: lazy create / update constraints → trip_id
  PT-->>BFF: phase trip_created

  Note over Loop: --- 发现环 (LLM OptA) ---
  PT->>Loop: act-or-stop (discover capability)
  Loop->>Loop: LLM 提名 20-30 名 (行程类型→venue 类别提示)
  Loop->>Loop: 清洗 (去重/规范化)
  loop 逐名 grounding
    Loop->>Reg: lookup (provider, native_id)
    alt 命中
      Reg-->>Loop: cached card (skip searchPlaces)
    else miss
      Loop->>Maps: searchPlaces(name, city)
      Maps-->>Loop: PlaceCard(s)
      Loop->>Reg: optional upsert (facts only, no must_see)
    end
  end
  Loop->>Trip: write trip.candidates (replace, 本 trip only)
  PT-->>BFF: phase candidates_ready

  Note over Loop: --- 骨架环 ---
  PT-->>BFF: phase skeleton_generating
  Loop->>Trip: read trip.candidates (NOT merge registry)
  Loop->>Loop: LLM 排骨架 (池 = grounded candidates)
  Loop->>Loop: 校验不修补 (deviations 可选)
  Loop->>Trip: commit ItinerarySkeleton + deviations
  PT-->>BFF: phase skeleton_ready + trip_id, revision

  BFF->>Fetch: fields [skeleton, constraints, deviations]
  Fetch->>Trip: read partitions
  Fetch-->>BFF: skeleton days[] + constraints + deviations[]
```

### 4. 数据结构（关键新增/变更）

```typescript
// trip.candidates — 本 trip only，registry 不并入
interface TripCandidates {
  tripId: string;
  candidates: PlaceCard[];      // grounded, 本 trip 提名
  // must_see 在 card 上是 per-trip 标记，不进 registry (ADR-056 D3)
}

// deviations — 校验不修补的透明产物（agent-discover-110c）
interface SkeletonDeviation {
  field: string;       // e.g. "far_cluster" | "attraction_pool" | "day_count"
  expected: string;
  actual: string;
  reason: string;
}

// plan_trip / make_itinerary skeleton (T3++Q 增量)
interface ItinerarySkeleton {
  days: SkeletonDay[];
  deviations?: SkeletonDeviation[];   // LLM 可附带；代码合并边界事实
}

// plan_trip response (T3++ 增量)
interface PlanTripResponse {
  trip_id: string;
  revision: string;
  status: "ready" | "planning" | "need_input" | "failed";
  phases: PhaseEvent[];
  // deviations 在 skeleton JSON 内持久化；2play-plan-103 从 fetch skeleton 读取展示
}
```

### 5. 起飞 11 → 提名 prompt 参数拼装审计（ADR-059 / 110a）

链路：where2play `toAgentPlanTripBody` → agent `PlanTripInput` → `buildNominateMustSeeUserMessage` trip 行。

| 起飞字段 | BFF body key | agent input | prompt prefs | trip 行标签 | 状态 |
| --- | --- | --- | --- | --- | --- |
| city | city | city | city | 无（首词） | OK |
| startDate | bounds.start | bounds.start | bounds.start | 日期段 | OK |
| days | numDays | numDays | numDays | `N天` | OK |
| partySize | party_size | party_size | party_size | `N人` | OK |
| budget | budget(mapped) | budget | budget | catalog 标签 | OK |
| tripType | trip_type | trip_type | trip_type | catalog 标签 | OK |
| pace | pace | pace | pace | catalog 标签 | OK |
| transit | transit_preference | transit_preference | transit_preference | catalog 标签 | OK |
| locale | locale | locale | locale | 决定语言 | OK |
| originName | origin.name | origin?.name | origin_name | `起点 XXX` | OK |
| startTime | start_time | start_time | start_time | `出发 HH:MM` | OK |
| mustInclude | must_include | must_include | must_include | `必去 XXX` | OK |
| other | other | other | other | `其他：XXX`（须带标签） | **须带标签** |

### 6. 能力合同（摘要）

> **110e 增量（骨架闸门软化 + 限制条件解释字典）：** pace 由硬配额（tight≤6/medium≤5/relaxed≤4）改为软语义提示；删 minAttr 硬下限（保留「池不足可少排」软提示）；午餐规则不再对单景点日静默重排；`attractionDwellMinutes` 增乐园/度假区档位（≥180min，venue-type 词汇，ADR-066，非城市 POI 名录 ADR-042）；新增 `buildConstraintGlossary` 向骨架 prompt 注入 11 条限制条件解释（i18n）。安全闸保留：跨日不重用、城市名不作站点、裸区域不作景点、stay 首位、schema 合法、must_include 覆盖。

| 能力 | 契约 |
| --- | --- |
| 发现 | LLM OptA 单条消息提名 20–30 名；行程类型→venue 类别提示（ADR-066，禁止城市→POI）；反模糊；季节硬规则 + 清洗 |
| 清洗 | 去重、规范化名称 |
| Grounding | 逐名 `searchPlaces`；registry 命中（`(provider, native_id)`）跳过 search；miss 可选 upsert（事实 only，无 `must_see`） |
| 候选池 | 写入 `trip.candidates`（replace）；骨架/enrich 只读本 trip，**不** `mergeRegistryPlaces` |
| 骨架 | LLM 排骨架（池 = grounded candidates）；`other` 标签去「偏好」 |
| 校验 | `ensureFarClustersOwnDays` 改校验只读；漂移写 `deviations`，不回改 |
| POI 不足 | 扩搜半径需 `need_input` 用户确认（`agent-discover-110d` / `2play-plan-104`） |
| 删除 | `skeletonPoolQueries` / `expandPlacesForSkeleton` / 模板搜主路径 |
| 观测 | `/debug/plan` 可读 trip.candidates + deviations；registry 仍按城市可读（cache-only） |

### 7. 故事切片

| Story | MVP | 范围 |
| --- | --- | --- |
| `agent-discover-110a` | **T3++**（本 MVP） | OptA 提名 → 清洗 → grounding → candidates（registry cache-only，不补池）；删骨架模板搜；ADR-059 |
| `agent-discover-110b` | T3++Q | 骨架提示 2a-none + other 去「偏好」标签 |
| `agent-discover-110c` | T3++Q | 校验不修补 + `deviations` JSON/DB（**Done** 2026-09-11） |
| `2play-plan-103` | T3++Q | Assistant 骨架下文字展示 deviations（i18n）（**Done** 2026-09-11） |
| `agent-discover-110d` | T3++Q | POI 不足扩半径 + `need_input`（**Done** 2026-09-11） |
| `2play-plan-104` | T3++Q | 扩半径用户确认 UI |
| `agent-discover-110e` | T3++Q | 骨架闸门软化（pace 软上限 / 删 minAttr 硬下限 / 午餐规则软化 / 乐园 dwell 档）+ 11 条限制条件解释字典 |

### 8. 非目标（T3++ = 110a only）

- 骨架 2a-none / deviations / 扩半径（→ **T3++Q**）
- ~~必去提名带理由 / 聊天 refine（→ T4，`agent-itinerary-101` / `2play-plan-102`）~~ — **T4 Cancelled by ADR-069**；chat refine 并入 T8
- `plan_next_stop` fill、meals、directions、四卡（→ T5+）
- 把 registry 当回发现源或候选池补池（显式禁止）
- per-city POI 百科 / CATALOG 扩展（ADR-042）
- intake `recoverPoolIfEmpty` 仍可保留模板 query（本切片仅禁骨架发现路径）

### 9. 合规

- **ADR-042**：LLM 提名非城市→POI 表；可有行程类型→venue 类别提示，禁止品牌/城市场名录。合规。
- **ADR-050**：LLM 在 places-agent，非 BFF。合规。
- **ADR-056**：upsert 语义保留；plan_trip 路径不再 merge registry 进候选池（cache-only amendment）。合规。
- **ADR-066**：提示可用 venue-type 类别（乐园/动物园…）；禁止城市→迪士尼类行。合规。
- **真智能体原则**：代码提供能力（grounding、validation），模型提供判断（提名、排程、deviation）。合规。

---

<a id="core-trip-store"></a>

## 21. Trip Store（PG 权威 + 内存热副本）+ 按需读取（MVP-16，ADR-046 Accepted 2026-09-02）

**真源：** [ADR-046](../adr/ADR-046-trip-store-pg-memory-fetch.md) · [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md) 批次 16（Feature **63–66**）。

### 21.1 目标与非目标

| 目标 | 说明 |
| --- | --- |
| G1 | 多工具改**同一份**行程（换餐、调序、受控改骨架，不必整次重跑 `make_itinerary`） |
| G2 | 宿主（尤其 where2play）更好使用行程数据与上下文 |
| 附带 | 减 MCP 大包 JSON；**非**为再抠 LLM 秒数 |

**非目标：** 宿主直连 DB；新增 `start_trip`；对外暴露 `patch_skeleton`；默认按日并发 LLM 骨架；恢复 MCP transport session 当业务状态。

### 21.2 存储与同步

- **权威：** PostgreSQL（§10 / ADR-025），薄表 + JSONB 分区字段可接受；忌长期上帝单列无版本。
- **热副本：** 进程内内存实体；**非**双主。
- **写：** 改内存 → 落 PG → 返回新 `revision`（可带 patch）。
- **读：** 内存命中；未命中或 revision 落后 → PG → 内存。
- **冲突：** 乐观锁 `revision`。
- **隔离：** `caller_key` + `expires_at`（TTL）；`trip_not_found` 时禁止宿主编造。

### 21.3 逻辑模型

```text
Trip
  id, revision, status, caller_key, locale, created_at, expires_at
  constraints   # city, bounds, pace, budget, origin, must_include
  candidates    # places[], restaurants[]（可 slim）
  skeleton      # days[] stop-order
  cursor        # day_index, stop_index
  filled[]      # per-stop slot, legs, notes, card refs
  artifacts[]   # kind=visa|weather|tips|… payload
```

### 21.4 `trip_id` 生命周期

- **懒创建：** 任一需账本的业务工具在**无** `trip_id` 时创建（PG + 内存），响应返回 `trip_id`；宿主后续必须带上。
- **不**新增 `start_trip` / `create_trip`。
- 调用顺序可变（discover / visa / tips / make 谁先谁建）。

### 21.5 工具面（目标态）

| 工具 | 角色 |
| --- | --- |
| `discover_places` / `make_itinerary` / `visa_requirement` / `travel_tips` 等 | 可懒创建；写各自字段；返回 `trip_id` + `revision` + patch/最小块 |
| `plan_next_stop` | **保留**写侧；吸收原 `display_current_stop` 的写/slot 职责（F65） |
| `fetch_trip_details` | **新增**只读：`trip_id` + `fields[]` |
| `display_current_stop` | **删除**（F65）；读改 fetch |
| `patchSkeleton` | **仅** `src/core` 内部；由 fill 等路径调用；不注册 MCP/HTTP |

写响应形状（目标）：`{ trip_id, revision, patch?, next_tool_call?, …最小展示块 }`。  
读：`fetch_trip_details` 按 fields 切片（constraints / skeleton / day_n / filled / artifacts / …）。

### 21.6 宿主镜像

| 宿主 | 策略 |
| --- | --- |
| **任何 HTTP 客户端** | **硬性：** 写后读行程必须 `POST /v1/fetch_trip_details`（`trip_id` + `fields[]`，可选 `day_index`）。写信封不得当骨架/池/约束/filled/artifacts 真源。不设 `info_id`。 |
| where2play | **MVP-18：** 每步写工具返回后 `fetch_trip_details`；本地 hydrate 以 fetch 切片为准。写响应里的 patch 可作乐观更新，冲突以 fetch 为准。 |
| Cursor / ChatBox | 弱镜像（`trip_id` + 摘要）；细节 fetch。MCP 可看写信封，**不**豁免 HTTP。 |

### 21.7 与历史 as-built 的关系

MVP-10～15 的 `display_current_stop` + 宿主回传 skeleton 链已归档至 [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md)。MVP-16 落地后：

1. 填充主路径改为 `trip_id` + cursor（candidates 不再每步回传）。
2. 链：`… → plan_next_stop → …`；展示用 `fetch_trip_details`。
3. 当前 live 路径见「MVP-T3」与「MVP-T3++」节；fill（`plan_next_stop`）推迟到 T5+。

### 21.8 分期

见 refactor-plan MVP-16：P0 = F63+F64+F66 评估；P1 = F65+精简落地；P2 = 内部 patch + artifacts；P3 = 清理/观测。

### 21.9 反模式

- 下发 `DATABASE_URL` 给 MCP 宿主
- 仅内存权威、无 PG
- 真双主无 revision
- 对外 `patch_skeleton` / 新增 `start_trip`
- 默认按日并发骨架冒充加速
- HTTP 客户端用写工具信封渲染行程事实（须 `fetch_trip_details`）

---

<a id="core-registry"></a>

## Registry（cache-only · [ADR-056](../adr/ADR-056-registry-backfill-semantics.md) / [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**角色（2026-09-11）：** POI 事实卡缓存，不是发现源。

| 规则 | 说明 |
| --- | --- |
| 身份 | `(provider, native_id)`（ADR-056 D1） |
| 命中 | grounding 跳过 `searchPlaces` |
| Miss | 可选 upsert；**仅事实**（无 `must_see`，ADR-056 D3） |
| 禁止 | `enrich` / skeleton **`mergeRegistryPlaces`** 把整城 registry 并入 trip 候选池 |
| 候选真源 | 本 `trip.candidates`（LLM 提名 + grounding） |

观测：`/debug/plan` 仍可按城市列 registry；空态诚实；**禁止**编造 POI（ADR-042）。

---

<a id="core-t3"></a>

## MVP-T3 — plan_trip skeleton-first（**Done** · usable Confirmed 2026-09-11）

**范围：** 起飞提交后创建 `trip_id`、骨架 make/commit、阶段信号；**不**固定四问；**不**必去提名；**不** `plan_next_stop` fill。见 [ADR-062](../adr/ADR-062-mvp-t3-skeleton-vs-t4-nominate.md)（切片）、[ADR-063](../adr/ADR-063-skeleton-only-plan-trip.md)、`agent-itinerary-100`、`2play-plan-101`。  
**发现路径：** **as-built** = 模板 stops-pool；**Target** = LLM 驱动发现（[ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md) · MVP-T3++ `110a`+）。  
**调用方 UI：** [`2play-design.md`](../2play-specs/2play-design.md) §4.7.1（按屏 + BFF 合同 + 时序图）。

### 1. 能力合同（摘要）

| 能力 | 契约 |
| --- | --- |
| 入参 | 起飞 11 边界（destination / tripType / budget / startDate / days / partySize / pace / transit / startTime / origin / other）；locale；**省略 `providers[]`**（ADR-052 区域路由） |
| T3 标志 | where2play T3 路径传 **`skeleton_only: true`**（见 [ADR-063](../adr/ADR-063-skeleton-only-plan-trip.md)）。MCP/全环调用方**省略**该标志 → 仍可走既有 intake / fill 全环，不变。 |
| 出参 | 非空 `trip_id` + `revision` + `status`；进度 `phase` 事件（终态 JSON 须带 `phases[]` 供 BFF 回放） |
| 读 | `fetch_trip_details` → `skeleton`（按日有序停点）+ `constraints` |
| 停止点 | make/commit **skeleton** 后停止；无 fill / meals / directions；`status: ready` **不**要求 `filledStops.length > 0` |
| 问卷 | `skeleton_only` 路径**不**返回固定 hotel/start_time/must_see/other `need_input` |
| 观测 | stops-pool / registry 按城市可读；空态诚实（ADR-042/056） |
| UI 文案 | 调用方用户可见「**框架**」；字段名仍 `skeleton` |

### 2. 详细技术设计

#### 2.1 入参归一（Takeoff11 → Trip constraints）

| Takeoff 字段 | Trip / plan_trip HTTP body / constraints |
| --- | --- |
| destination (+ geocode 标签) | `city` + 城市锚点 lat/lng/country/city(/city_en) |
| startDate + days | `bounds.start` / `bounds.end` 或等价；`numDays` |
| partySize | **`party_size`**（须进 schema → constraints，不得剥掉） |
| tripType / budget / pace / transit | `trip_type` / `budget` / `pace` / `transit_preference` |
| origin | `origin.name`（+ lat/lng 若已验真） |
| startTime | **`start_time`** → constraints（每日默认出发时间） |
| other | **`other`** → constraints（可空自由文本） |
| （T3） | **`skeleton_only: true`** |

**ADR-059：** 已知非空条件（含 `start_time` / `other` / `party_size`）不得在骨架提示/入参/持久 constraints 中丢弃。  
**ADR-052：** 省略 `providers[]` → 大陆 AMAP-only；HK 双源；其余 Google；不按 CJK/locale 扩源。

#### 2.2 编排停止策略（T3）

```text
plan_trip(T3 mode / where2play after-submit)
  → geocode / resolve destination if needed
  → lazy create trip_id；persist constraints
  → emit phase: trip_created
  → (optional) search_places to seed pool / registry backfill
  → emit phase: skeleton_generating
  → make_itinerary / commit skeleton (internal)
  → emit phase: skeleton_ready
  → STOP  // 禁止 plan_next_stop；禁止强制四问 need_input
```

与 T1 as-built 差异：T1 可在边界不全时返回四问 `need_input`；**T3 where2play 路径**假定起飞 11 已齐（must-see 故意不齐也不发 Q3）。MCP/ChatBox 其它入口仍可 agent-driven ask（非本切片 where2play 产品路径）。

#### 2.3 Phase 事件合同（调用方可映射 i18n）

| `phase` | 含义 | 最低载荷 |
| --- | --- | --- |
| `trip_created` | Trip 已持久 | `trip_id` |
| `skeleton_generating` | 正在写骨架 | `trip_id` |
| `skeleton_ready` | 骨架可 fetch | `trip_id`, `revision` |
| `failed` | 不可恢复失败 | `trip_id?`, `error.key` |

传输：HTTP 优先 **NDJSON**（或 SSE）推送 `phase`；若实现仅终态 JSON，须在单响应内带齐已发生 phase 列表或等价进度字段，供 BFF 回放。**禁止**依赖 2play 本地 LLM 生成进度散文。

#### 2.4 `fetch_trip_details`（T3 最小 fields）

| field | T3 UI 用途 |
| --- | --- |
| `constraints` | 主区只读出行限制 |
| `skeleton` | 主区 Day tabs + `.slot--skeleton`；助手 route-spine |

不要求 T3 返回 `filled` / `artifacts`（贴士 → T7）。空 skeleton 或 stay-only 视为失败（与既有骨架质量闸一致）。

#### 2.5 Registry / stops-pool

合格景点 commit 后按 [ADR-056](../adr/ADR-056-registry-backfill-semantics.md) 回填。`/debug/plan` 与观测读：按目的地城市列 pool；空池 → 明确空态；**禁止**编造 POI（ADR-042）。

> **2026-09-11 amendment（ADR-067）：** T3 as-built 仍走模板池；T3++ 起 registry 退化为 **cache-only** grounding 加速器 —— `enrich` / skeleton **禁止** `mergeRegistryPlaces` 把整城 registry 并入 trip 候选池。Trip candidates 只来自 LLM 提名 + grounding 的本 trip 名单；registry 命中可跳过 `searchPlaces`，miss 时可选 upsert（仅事实，无 `must_see`）。详见上文「MVP-T3++」与「Registry（cache-only）」。

#### 2.6 错误

| 情况 | `error.key`（示例） | 调用方 |
| --- | --- | --- |
| 供应商/模型骨架失败 | `errors.skeleton_failed` / 既有 make 失败键 | i18n；不写假 filled |
| 缺 key / 未授权 | 既有 auth 键 | 不泄露密钥 |
| 超时 | `errors.plan_timeout` 等 | 可重试 |

### 3. 时序图 — agent 内侧与 2play 协作

与 2play §4.7.1 §C 对齐；本图强调 agent / Trip / vendors：

```mermaid
sequenceDiagram
  autonumber
  participant BFF as where2play BFF
  participant PT as plan_trip tool
  participant Loop as Agent tool loop (Qwen)
  participant Trip as Trip Store
  participant Reg as POI registry / pool
  participant Maps as AMAP / Google
  participant Fetch as fetch_trip_details

  BFF->>PT: plan_trip(Takeoff11, locale)
  PT->>Trip: create/update constraints → trip_id
  PT-->>BFF: phase trip_created

  PT->>Loop: act-or-stop (skeleton-first system/tool surface)
  Loop->>Maps: search_places / geocode (as model decides)
  Maps-->>Loop: PlaceCards
  Loop->>Trip: commit candidates / constraints patches
  Loop->>Reg: safeUpsertEligiblePois (ADR-056)
  PT-->>BFF: phase skeleton_generating

  Loop->>Trip: make/commit ItinerarySkeleton
  Note over Loop,Trip: STOP before plan_next_stop
  PT-->>BFF: phase skeleton_ready + trip_id, revision

  BFF->>Fetch: fields [skeleton, constraints]
  Fetch->>Trip: read partitions
  Fetch-->>BFF: skeleton days[] + constraints

  opt debug
    BFF->>Reg: list city stops-pool
    Reg-->>BFF: entries or empty
  end
```

### 4. 非目标（T3）

- 固定四问 `need_input`（where2play）
- ~~必去提名带理由 / 聊天 refine（→ T4）~~ — **T4 Cancelled by ADR-069**；chat refine 并入 T8
- `plan_next_stop` fill、meals、directions、artifacts 四卡全量（→ T5+）
- what2eat 工具并入 `plan_trip`（ADR-050 D3）

~~T4（提名+聊天 refine）~~ 见 ~~`agent-itinerary-101` / ADR-062~~ — **Cancelled by ADR-069**。chat refine 并入 T8（`agent-chat-93e` / `2play-plan-90e`）。

---

<a id="core-mcp-hosts"></a>

## 第三方工具调用（Cursor / ChatBox）

**定位：** Cursor、ChatBox 等 MCP 宿主用**宿主自己的模型**选工具；places-agent 只提供工具核心（与 HTTP `/v1` 同一套函数）。**不**要求终端用户改宿主 system prompt（[ADR-040](../adr/ADR-040-plan-itinerary-align-split-tools.md) D3/D4'）。与 where2play 的差异：2play 走 BFF HTTP + 零产品 LLM（[ADR-050](../adr/ADR-050-where2play-no-product-llm.md)）；MCP 宿主可直接自然语言命中工具。

**真源补充：** [`mcp-client-integration.md`](../knowledge/agent/mcp-client-integration.md) · §8 MCP · [ADR-003](../adr/ADR-003-dual-transport.md) · [ADR-016](../adr/ADR-016-custom-http-server.md)。

### 1. 宿主对照

| 宿主 | 传输 | URL | 认证 | 对话模型 |
| --- | --- | --- | --- | --- |
| **Cursor** | Streamable HTTP | `…/mcp` | `Authorization: Bearer <caller_key>` | Cursor 宿主模型 |
| **ChatBox** | Legacy SSE | `…/sse`（+ `/messages`） | 同上 | ChatBox 宿主模型 |
| **where2play** | HTTP `/v1` | BFF → `plan_trip` / `fetch_trip_details` | Bearer（服务端） | **无**产品 LLM（ADR-050） |

- 协议 id：`serverInfo.name` = **`places-agent`**（字面量，非 i18n；[ADR-013](../adr/ADR-013-caller-agent-id.md)）。
- 工具描述须含字面量 `places-agent`，降低宿主误选通用搜索 / 地图 MCP。
- **禁止**把 MCP transport session 当业务状态；行程真源是 Trip Store（`trip_id` + `revision`）。
- ChatBox **不要**指向 `/mcp`；Cursor **不要**用 stdio `command`/`args`（远程用 `url` + `headers`）。

### 2. 数据流（MCP 宿主 ↔ places-agent）

```mermaid
flowchart LR
  subgraph Host[第三方宿主]
    UM[用户自然语言]
    HM[宿主模型<br/>Cursor / ChatBox]
  end
  subgraph Agent[places-agent 同进程]
    MCP["/mcp 或 /sse"]
    CORE[Tool core<br/>与 /v1 同源]
    TRIP[(Trip Store)]
    REG[(Registry cache-only)]
    MAPS[AMAP / Google]
  end

  UM --> HM
  HM -->|tools/call Bearer| MCP
  MCP --> CORE
  CORE --> TRIP
  CORE --> REG
  CORE --> MAPS
  CORE -->|tool result + host_instructions| HM
  HM -->|展示 / 追问| UM
```

### 3. 时序图 — 真智能体路径（`plan_trip`）

MCP 宿主与 2play 共用对外两方法；**省略** `skeleton_only` 时可走 intake / 全环（与 where2play T3 路径不同）。

```mermaid
sequenceDiagram
  autonumber
  participant User as 用户
  participant Host as Cursor / ChatBox<br/>宿主模型
  participant MCP as places-agent MCP
  participant Core as Tool core
  participant Trip as Trip Store

  User->>Host: 自然语言（城市 / 天数 / 偏好）
  Host->>MCP: tools/call plan_trip (Bearer)
  MCP->>Core: 同 HTTP 契约
  Core->>Trip: lazy create / patch → trip_id
  alt 边界不全
    Core-->>Host: need_input + question（agent 出题）
    Host-->>User: 展示问题
    User->>Host: 回答
    Host->>MCP: plan_trip(trip_id + 补丁)
  else 边界齐
    Core->>Core: 发现 / 骨架（T3++ Target：LLM 提名+grounding）
    Core->>Trip: commit skeleton + candidates
    Core-->>Host: trip_id, revision, status, phases
  end
  Host->>MCP: fetch_trip_details(fields)
  MCP->>Trip: read partitions
  MCP-->>Host: skeleton / constraints / …
  Note over Host: 弱镜像：可看写信封；行程真源须 fetch<br/>禁止编造 POI / 坐标
  Host-->>User: 按日展示骨架
```

### 4. 配置

**Cursor**（`.cursor/mcp.json` 或 Settings → MCP）：

```json
{
  "mcpServers": {
    "places-agent": {
      "url": "https://places.agent-mate.ai/mcp",
      "headers": {
        "Authorization": "Bearer ${env:PLACES_AGENT_CALLER_KEY}"
      }
    }
  }
}
```

本地：`http://localhost:<PORT>/mcp`（与 `.env.local` 的 `PORT` 一致）。改配置后 Reload。

**ChatBox**（Remote HTTP/SSE）：

| 字段 | 值 |
| --- | --- |
| Type | Remote (HTTP/SSE) |
| URL（prod） | `https://places.agent-mate.ai/sse` |
| URL（local） | `http://localhost:<PORT>/sse` |
| Header | `Authorization: Bearer <caller_api_key>` |

连接名用 **`places-agent`**。工具按**会话**注入：改 MCP 后须**新开对话**并打开该 server；旧线程不会自动挂上工具。允许多 MCP 并存（ADR-040 D7）；靠工具描述把行程意图路由到 places-agent，**不要**要求运营关掉其他 MCP。

### 5. 宿主契约

| 规则 | 说明 |
| --- | --- |
| 对外方法 | 真智能体路径：**`plan_trip`** + **`fetch_trip_details`**。内部意图（收边界 / 建骨架 / 填细节）**不**注册为 MCP 工具。 |
| 弱镜像 | 可看写信封；骨架 / 池 / constraints / filled / artifacts 真源 = `fetch_trip_details`（与 HTTP 一致）。 |
| 不编造 | 工具失败或空池时，宿主**不得**用参数知识编造行程/地点/坐标；遵循 `host_instructions`。 |
| 无 system-prompt 门 | 产品路径靠工具 `need_input` / `host_instructions` / description；**禁止**把「用户必须改 ChatBox system prompt」当交付条件。 |
| `skeleton_only` | where2play T3 传 `true`；MCP/ChatBox **省略** → 可 intake / 全环，不挡 T3++ 发现能力。 |
| 密钥 | Bearer = 运营者后台签发的 caller key；**绝不**把地图 / Qwen / `DATABASE_URL` 下发给宿主。 |

### 6. 非目标（第三方 MCP）

- 以 MCP transport session 持久化行程
- 要求用户改宿主 system prompt 才能规划
- 宿主直连 DB / 读 registry 当发现源
- 把 Google Worker MCP（`GMAPS_MCP_*`）暴露成客户端工具（那是 places-agent **服务端** Google 回退）

---

<a id="core-principles"></a>

## 真智能体 — 能力与原则

### 能力清单

MVP 切分依据；每行组合见 [`product-backlog.md`](../product-backlog.md) §1 `true-agent` / `TA`。

| 能力 | 对外/内部 | 输入 | 产物 | 关联 ADR |
| --- | --- | --- | --- | --- |
| `plan_trip` | 对外（MCP + HTTP） | 行程边界 / `trip_id`+补丁 | `trip_id`, `revision`, `status`, `need_input` | 050 / 060 |
| `fetch_trip_details` | 对外 | `trip_id`, `fields[]`, 可选 `day_index` | `skeleton` / `filled` / `candidates` / `constraints` / `cursor` / `artifacts` | 046 / 051 |
| `geocode` | 内部 | 城市 / 地址 | 锚点坐标（WGS84） | 052 / 048 |
| `search_places` | 内部 | 名 / 类目（省略 `providers[]`） | PlaceCard 入 stops pool（仅景点） | 052 / 049 |
| `directions` | 内部 | 站间坐标 | ETA / 腿（权威时长只来自供应商） | 052 |
| `commit_trip` | 内部 | 声明式补丁 | 升 `revision` | — |
| `ask_user` | 内部（可选） | 缺约束 | `need_input`，不猜 | — |
| 必去芯片 | `plan_trip` intake | 城市锚点 | `candidates` 入池（**不保证**非空，ADR-060；**无 must_see 标志**，ADR-069） | 051 / 060 / 069 |
| 四卡 | `plan_trip` 全环（T4） | 目的地 + 起止日 | `artifacts.tips` / `artifacts.visa` | 014 / 044 |
| chat 改行程 | `plan_trip` **refine 模式**（MVP-T9 `agent-chat-93e`） | `trip_id` + `refine.instruction` | 模型在已有 Trip 上选 `commit_trip.operations[]` 补丁/重排；返回 `reply` + `itinerary` | 050 / 93e |

**不在本表（what2eat，ADR-050 D3）：** `search_restaurants`、2eat `chat` / `geocode` / `get_place_details` 不并入 `plan_trip`。

### 原则

循环在 places-agent 进程内：模型看行程目标 + 已有 Trip + 可用工具，决定 act 或停。

- 宿主 **只**调对外两个方法，不按 discover → make → plan_next_stop 调度。
- 「收边界 / 建骨架 / 填细节」是环的内部意图，**不是**对外 API；**不**注册 `intake_*` / `compose_*` MCP。
- 真智能体 = 谁持有循环；事实闸仍由代码执行。
- **where2play 零产品 LLM（ADR-050）：** BFF 只渲染 `need_input`、回传答案、HTTP 写/读。
- **起点整卡（ADR-053）；搜源省略 `providers[]`（ADR-052）。**

| 宿主 | 行为 |
| --- | --- |
| **2play HTTP** | `plan_trip` + `fetch_trip_details`；展示 agent 下发问题与芯片 |
| **Cursor / ChatBox（MCP）** | 自然语言命中 `plan_trip`；细节 fetch；传输与配置见 [第三方工具调用](#core-mcp-hosts) |
| **what2eat** | **不在行程方案范围** |

### 对外方法（两个）

**`plan_trip`：** 收行程边界或 `trip_id`+补丁。城市齐 → 懒创建 trip、intake 环尽量出芯片、`need_input`（问题由 agent 给）。边界齐 → 全环排程（T2+）。回 `needs_input` \| `planning` \| `ready` \| `failed`。HTTP 写响应不当行程真源。别名：`plan_itinerary` / `trip_plan` / `trips`。

**`fetch_trip_details`：** `fields[]` 含 `skeleton` \| `candidates` \| `constraints` \| `filled` \| `cursor` \| `artifacts`。读路径不解析图、不升 `revision`（ADR-046 / ADR-051）。

细化检查表：[`../knowledge/agent/real-agent-refinement-checklist.md`](../knowledge/agent/real-agent-refinement-checklist.md)。

### 能力逻辑与提示组合（内部意图）

下表中的能力是 agent 环的**内部意图**，**不是**对外 API，也**不**注册为 MCP 工具。模型在环内看「行程目标 + 已有 Trip + 可用工具」决定 act 或停；每项能力的**事实闸由代码执行**（坐标、池归属、must_include 覆盖、geo 边界）。提示组合只负责把用户输入与 Trip 状态映射成模型可读的指令，**不**承担事实正确性。

> 命名约定：对外 `plan_trip` / `fetch_trip_details`；内部意图用中文名（收边界 / 必去提名 / 建骨架 / 填细节 / 四卡）。实现侧函数名见 `places-agent/src/core/`（`plan-trip.ts` / `itinerary-planner.ts` / `make-itinerary.ts` / `plan-next-stop.ts` / `trip-artifacts.ts`）。

#### 通用提示组合骨架

每个能力的 user message 由以下片段按需拼接（顺序固定，缺失片段省略，不编造）：

1. **任务句**：能力目标 + 产物形状（如「只排停点顺序，不出时间/交通」）。
2. **硬约束**：池归属、must_include 覆盖、pace 上限、meal 节奏、不编造坐标/名称。
3. **用户输入映射**：从 Takeoff11 / `constraints` / `need_input` 答案取字段，逐项落成自然句或键值行；空值省略，不补猜。
4. **候选/上下文**：候选池行、origin、已 committed 的 must_see 芯片、上一轮 fill 游标等。
5. **输出契约**：JSON schema 描述 + locale + 「只返回 JSON」。

系统提示由 `assembleSystemPrompt({ locale, intent, budget, glossary })` 组装（base + overlay + glossary），**不**含城市 POI 知识（ADR-042）。

---

#### 1. 收边界（intake）

| 项 | 内容 |
| --- | --- |
| 触发 | `plan_trip` 城市齐但边界缺；MCP 自然语言命中 `plan_trip` |
| 输入 | 已知 constraints（city / origin / dates / party_size / trip_type …） |
| 逻辑 | 模型先 `geocode` 锚点 → `search_places` 出 must_see 芯片 → `ask_user` 一次问齐缺项（hotel / start_time / must_see / other）；不猜、不强制 L3（ADR-060） |
| 提示组合 | system = `systemPrompt(city, locale)`（含 placesOntology + 工具顺序 1–6）；user = 已知边界摘要 + 「Hold the loop. Call tools until chips are committed, then stop.」 |
| 事实闸 | 芯片名必须来自 `search_places` 命中；`commit_trip` 只收搜到的名；无有限 lat/lng 不入池 |
| T3 不走 | where2play T3（`skeleton_only`）**不**触发收边界四问（起飞 11 已齐，must-see 故意不齐也不发 Q3，ADR-062） |

#### 2. 必去提名（nominate must-see / discovery）

> **ADR-069 amendment (2026-09-11 · 再确认 2026-09-14):** must_see 标记层（C）已删除。提名能力保留为发现路径（A，110a），不再产出 `must_see` 标志或理由文本。T4 必去理由交互 Cancelled；chat refine 并入 T8。**产品政策：不再做 agent 提示必去点相关功能**（不恢复 C/D、不新开必去标记/理由/必去 UI）。

| 项 | 内容 |
| --- | --- |
| 触发 | 发现路径（110a OptA）；MCP 全环；**非** T3 skeleton_only |
| 输入 | city / numDays / limit / 现有池 / trip_type / pace / budget / party_size / transit_preference / bounds / origin_name / must_include / other |
| 逻辑 | LLM 产出**短地名**（不排行程、不出坐标）→ 代码 `groundNominatedName`/`hydrateNominatedCard` 用 `suggest_places`+`search_places` 把每个名字落到真实 PlaceCard → 入池；落不下的丢弃（事实闸） |
| 提示组合 | `buildNominateMustSeeUserMessage(city, limit, numDays, prefs)`：任务句「提名公认必去，不安排行程」+ 硬约束（短地名、无括号、不编造坐标、近郊至少一个、同一片区域成簇）+ 用户输入行（trip_type 显示名 / pace / budget / party_size / transit / origin / must_include / other）+ JSON 输出契约 |
| 事实闸 | 名字经 grounding 才入池；无城市 CATALOG（ADR-042） |
| 产物 | `candidates[]`（grounded PlaceCard，无 `must_see` 标志 — ADR-069 删除） |

#### 3. 建骨架（plan trip skeleton / `make_itinerary`）

| 项 | 内容 |
| --- | --- |
| 触发 | 边界齐 → 全环；where2play T3 `skeleton_only=true` 建骨架后停 |
| 输入 | city / numDays / candidates / origin / pace / budget / must_include / **结构化 prefs**（trip_type · party_size · transit · start_time · other · bounds）/ locale — **禁止**仅用 slug 拼成一条 `natural_language` 糊墙 |
| 逻辑（**as-built T3++Q · 110c**） | （1）LLM OptA 提名 → grounding → trip.candidates；（2）LLM 排骨架；（3）**校验不修补**：`ensureFarClustersOwnDays` 只读检测远簇 → `deviations[]`；（4）薄池 / day_count 边界事实写入 deviations；（5）commit skeleton（含 deviations） |
| 逻辑（**as-built T3/T3+** · 历史） | （1）`expandPlacesForSkeleton` / `skeletonPoolQueries` 模板搜 → `search_places` + eligible；（2）`enrichMakeItineraryInput`；（3）`buildSkeletonUserMessage` → LLM 骨架；（4）`ensureFarClustersOwnDays` 静默修补；（5）validate → commit |
| 逻辑（**Target T3++ · [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)**） | （1）LLM OptA 提名 20–30 名 → 清洗 → searchPlaces grounding（registry **cache-only**：命中跳过 search，**不补池/不 merge 整城库**）→ 写本 trip.candidates；（2）删模板搜路径；（3）LLM 排骨架（池=本 trip grounded）；（4）**校验不修补** + 可选 `deviations`；（5）commit。实现故事：`agent-discover-110a`→`110d` |
| 提示组合 | as-built：任务句 + 旅人块（含季节软规则）+ 候选行。Target：发现用 OptA 单条消息；骨架 2a-none（无季节规则段）；other 无「偏好」标签 |
| 事实闸 | 停点名 ∈ 池；must_include 覆盖；跨日唯一；pace；meal。Target：不静默改天数；不符合项进 deviations |
| 停止 | T3：commit 骨架后 `stopAfterSkeleton` |
| 质量切片 | as-built `102`–`109` Done；Target `110a`–`110d` + `2play-plan-103`/`104` |

**旅人块字段（空则省略）：** ISO dates（bounds）· month+season · season rule · trip_type 显示名或自定义原文 · party_size · budget 显示 · transit（软）· pace · origin · start_time（软）· Other（preference）· kids 池内排序句（软）。

#### 4. 填细节（plan trip details / `plan_next_stop`）

| 项 | 内容 |
| --- | --- |
| 触发 | 骨架 ready 后全环；T5+；**非** T3 |
| 输入 | cursor(day_index, stop_index) / current_stop / next_stop / candidates / city / anchor / transit_preference / pace / budget / time_from / stay_role / day_stops |
| 逻辑 | `skeletonFillHandoff` 出下一停游标 → `planNextStopFill`：directions/启发式出 ETA + slot 时段 + 餐档现搜 → patch 当日骨架 → 推进 cursor 至 `trip_complete` |
| 全环 stop 策略（**MVP-T5 S1 · A+B**） | 模型仍自主选工具；`stop` 工具描述 + `buildFullLoopSystemPrompt` 要求：**仅当** `plan_next_stop` 返回 `trip_complete`（全部非 stay 骨架站已填）后才可 `commit_artifacts` → `stop`。禁止部分填充后早停。实现：`FULL_LOOP_STOP_TOOL_DESCRIPTION`（`plan-trip.ts`）。探针：上海/杭州/里斯本 fill 100%。知识：[`full-loop-early-stop-ab.md`](../knowledge/agent/full-loop-early-stop-ab.md) |
| HTTP `answers` 续跑（**MVP-T5 TD-4**） | 同 `trip_id` 回传：`answers.expand_radius`（已有，110d）；**`answers.hotel`**：非空店名 → 设 `origin.name` 后继续全环；`"skip"` / `"__skip__"` / `""` → 定居宿题且不设起点，走 `stopAfterSkeleton`（骨架，非无起点满填）。dispatch 须转发 `hotel`，不得只留 expand_radius。 |
| `resolve_origin_stay`（**MVP-T5 TD-5**） | `pickLodgingStayCard`：名称无交叉脚本匹配时，若搜索仅命中 **1** 张 lodging 卡则采纳（EN 查询 × CN Google 标题，如东京蒙特利）。失败时 agent/legacy 共用 `nameOnlyOriginStay`（默认 `GOOGLE_MAPS` + city anchor）；工具 **once-guard**（已结算则不再搜）。禁止为单城加酒店表（ADR-042）。 |
| 提示组合 | 以 fill 输入为结构化上下文（非自由 prompt）；权威时长只来自 directions/启发式，**不**让模型编 duration |
| 事实闸 | 时长只来自供应商/启发式；餐店来自 `search_restaurants` 命中；不编造坐标 |

#### 5. 四卡（artifacts / tips + visa）

| 项 | 内容 |
| --- | --- |
| 触发 | 全环末 `commit_artifacts`；T7 |
| 输入 | destination + bounds(起止日) |
| 逻辑 | `travel_tips` adapter 一次 tips-prose + visa adapter → 写 `artifacts.tips` / `artifacts.visa` |
| 提示组合 | tips overlay（`prompts/overlays/travel-tips.md`）+ destination/bounds；不编造政策，visa 以 adapter 为准 |
| 事实闸 | visa 走 adapter 不走 LLM 编造；tips 不含城市 POI 百科 |

---

### 必去提名 vs 建骨架 — 关系决策（[ADR-065](../adr/ADR-065-nominate-vs-skeleton-relationship.md) → **superseded for discovery by** [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)）

**Accepted（as-built T3+）：** 提名独立能力 + 骨架 geo 多样性闸（B+C）。提名 → `candidates[].must_see=true` + 理由入池；骨架把提名当偏好消费；geo 闸用 haversine + 候选坐标，不按城市查表（ADR-042）。  
**Superseded（discovery）：** ADR-067 起，发现路径改为 LLM 驱动提名 + grounding，取代代码模板 stops-pool。T3+ Done：`102`–`109`；T3++：`110a`–`110d`；~~T4：`agent-itinerary-101` / `2play-plan-102`~~ — **Cancelled by ADR-069**。详见上文 [智能体规划行程核心机制](#core-t3pp)。  
**Superseded（must_see marking + T4）：** ADR-069 删除 must_see 标记层（C）+ T4 必去理由交互（D）。提名能力（A）保留为发现路径；must_include 硬闸（B）保留。

---

## 1. 目标与非目标

| 目标 | 非目标 |
| --- | --- |
| 一个 Node 进程：Next.js 管理后台 + `/v1` 工具接口 + MCP | 第四个 Portainer 栈或 MCP 边车 |
| 调用方看到的 id 为 `places-agent` | 以主机名作为 agent id |
| HTTP 和 MCP 使用相同的工具函数 | 分叉的仅 MCP 业务逻辑；将 Google Worker MCP 作为 `providers[]` id |
| 工具接口使用调用方 API 密钥；管理后台使用会话 Cookie | 以管理员 Cookie 授权地图工具 |
| 简单的 OPENAI_CN 工具循环 | Kubeflow、特征存储、按供应商分立的 LLM 智能体 |
| PostgreSQL 管理用户/密钥（[ADR-025](../adr/ADR-025-places-agent-postgres-prisma.md)） | 以 JSON 文件作为数据源；SQLite 卷（ADR-015）；共享 `what2eat` |
| 四种语言目录 | OpenCC；HK↔TW 回退；`next-intl` `[locale]` 路由 |

---

## 2. 运行时形态（单进程）

**入口：** 轻量自定义 Node HTTP 服务器（[ADR-016](../adr/ADR-016-custom-http-server.md)）。MCP SDK 原生使用 Node `IncomingMessage`；ChatBox SSE 为长连接。`CMD node server.ts`（Next `output: "standalone"`）。`next start` 不是入口。

```text
node server.ts  (PORT, one process)
  /mcp              → Streamable HTTP MCP (Bearer)
  /sse + /messages  → MCP SSE (ChatBox) (Bearer)
  *                 → Next.js
        / /login /admin/api-keys /admin/users /instructions   operator HTML
        /api/admin/*                           admin BFF (cookie)
        /v1/*  /v1/health                      HTTP tools (Bearer; health is public)
```

| 方案 | 结论 |
| --- | --- |
| 自定义服务器 + Next | **选用** |
| 仅用 Next 路由处理器承载 MCP | 拒绝 — SSE/Web `Request` 与 MCP Node 传输层不兼容 |
| MCP 边车 | 禁止（ADR-012） |

自定义服务器只对 `/mcp`、`/sse`、`/messages` 做特殊处理。`/v1` 保留为 Next 路由处理器，以便 REST、Zod 和 Vitest 继续使用 App Router。

**路径映射**（HTML 对应下文 §12）：

| 路径 | 认证 | 用途 |
| --- | --- | --- |
| `/login` `/login/fresh` `/reset-password` `/set-password` `/accept-invite` `/instructions` | 公开（会话可选） | 运营者 HTML；`/login/fresh` 清除会话后重定向至 `/login` |
| `/admin` `/admin/api-keys` `/admin/api-keys/new` `/admin/api-keys/[id]` `/admin/users` | 管理员会话 | 运营者 HTML；未登录 → `/login`。`/admin` 重定向至密钥页。 |
| `/api/admin/*` | 会话 + CSRF | 同源 BFF |
| `/v1/*` | Bearer 调用方密钥 | HTTP 工具接口 |
| `/v1/health`（以及 `/health` 别名） | 无 | `{ "agent": "places-agent", "ok": true }` |
| `/mcp` | Bearer | Streamable HTTP MCP |
| `/sse` + `/messages` | Bearer | ChatBox 使用的旧版 SSE |

Next 中间件匹配器：仅匹配 `/admin`、`/admin/*`、`/api/admin`、`/set-password`。**绝不**将会话中间件挂载至 `/v1`、`/mcp`、`/sse`、`/messages`。密码为空 → `/set-password`。已登录用户访问 `/login` → `/admin/api-keys`。

工具名称不加前缀。身份标识为 `serverInfo.name` 和 JSON 字段 `agent`。

---

## 3. 模块结构

```text
places-agent/
  Makefile
  server.ts                      # MCP dispatch + Next
  middleware.ts                  # cookie gate for admin HTML/API only
  prisma/schema.prisma
  prisma/seed.ts
  messages/{EN,CN,HK,TW}.json
  prompts/chat/v1.md
  prompts/glossaries/travel.v1.json
  app/                           # pages + /api/admin + /v1
  src/
    core/                        # tools, gateway, loop, t(), truncate
    adapters/{amap,google,tripadvisor,open-meteo}/
    mcp/                         # McpServer + transports
    http/                        # envelope
    auth/{session,caller,csrf}.ts
    db/                          # Prisma client + seed
```

**依赖方向：** `app` / `mcp` / `http` → `core` → `adapters`。**core 不得引入 Next、MCP SDK 或 Prisma。**

Next.js **16.3** App Router，React **19**，TypeScript **7**，Tailwind **4**，React Query，RHF + Zod。锁定 `@modelcontextprotocol/sdk` 和 `openai@6` 版本。实现时以当前 SDK 文档核实 MCP 注册 API。

---

## 4. 拓扑：单一执行核心，双入口路径

只有**一个认知任务**。规划器+工作节点方案以及按供应商分立的 LLM 智能体均无法满足延迟预算。管理后台是 CRUD，不是智能体。

```text
BFF / MCP host
 ├─ Direct tool HTTP/MCP ──┐   ← no LLM (caller already named the tool)
 └─ NL chat → OPENAI_CN loop ─┤
                            ▼
                     Tool core (one)
                            │
         ┌─────────┬────────┼─────────┐
         ▼         ▼        ▼         ▼
       AMAP     Google   Tripadvisor  Open-Meteo
                REST→MCP   enrich      helper
```

**自然语言循环（仅 Feature 10）：**

```
LOOP:
  Model sees: messages + locale + 5 tools (searches, details, navigate, geocode)
  Model decides: tool call or final answer
  If tool: run core, truncate, append, continue
  If answer: Layer A/C catalogs (incl. weather.wmo.*) + Layer L glossary if HK/TW
```

`plan_itinerary` 为 **HTTP/MCP** 接口。如果对话需要生成行程，调用同一函数（第 2 级进度列表）。只有在截断后的行程 JSON 仍然撑爆上下文时才将其隔离为子智能体 — **而非**按供应商拆分。

相信模型会自行排序 geocode → search → details。不要硬编码该顺序。

| 级别 | 用途 |
| --- | --- |
| 1 — 工具 | 自然语言对话默认 |
| 2 — 短进度列表 | 多步行程对话丢失边界/偏好时 |
| 3 — 单一行程子智能体 | 仅在可量化的上下文爆炸时使用 |
| 评估器-优化器 | 推迟到存在 HK/TW 黄金评估集之后 |

**上下文：** 截断供应商 JSON（保留 id、name、location、rating、hours、`sources[]`、跳过原因）。如果某张卡片无法保留 **id + 来源信息**，则以原因 key 标记该结果为失败。仅在 locale 为 `HK`/`TW` 时使用术语表。节奏规则仅在 `plan_itinerary` 轮次中生效。

**上传：** 提取简短结构化提示。不保留字节数据。不将 OCR 结果粘贴到系统提示中。上传失败 → 带 key 的错误；**不得从失败的上传中提取 POI**。图像生成不是 MVP 工具。

**失败处理：** 跳过 + 原因 key，**不允许静默替换**。Google 直连失败 → Worker MCP，来源标注 `GOOGLE_MAPS`。Tripadvisor 失败 → 省略富化数据。天气失败 → 降级行程，不清空行程。搜索为空 → 空列表 + key。OPENAI_CN 失败 → 返回错误，绝不返回空成功响应。幂等工具：**重试一次**后跳过；模型可基于**部分**结果作答。

**人机交互（HITL）：** 搜索/详情/地理编码/导航/对话均无需人工干预。仅在后续出现不可逆副作用时才添加审批节点。

**可观测性（MVP 日志）：** `trace_id`、工具名、`providers[]`、供应商延迟、跳过原因、locale、`prompt_id` / 术语表 id；每个 LLM 轮次：模型名 + token 数。

---

## 5. 工具核心

HTTP `/v1` 和 MCP 调用**相同的函数**。传输层负责认证、解析与封装。

| 函数 | HTTP + MCP 名称 | 自然语言循环 |
| --- | --- | --- |
| `searchRestaurants` | `search_restaurants` | 是 |
| `searchPlaces` | `search_places` | 是 |
| `suggestPlaces` | `suggest_places` | 是（可选 adapter；无则空列表） |
| `getPlaceDetails` | `get_place_details` | 是 |
| `navigate` | `navigate` | 是 |
| `geocode` | `geocode` | 是（必须保持公开） |
| `visaRequirement` | `visa_requirement` | 否（结构化；Orizn REST，ADR-044） |
| `planItinerary` | `plan_itinerary` | 仅当对话请求生成行程时 |
| `discoverPlaces` | `discover_places` | 否（结构化拆分） |
| `arrangeDay` | `arrange_day` | 否（结构化拆分） |
| `getWeather` | 非公开 | 行程辅助函数 |

面向调用方的核心公开工具为上表 HTTP+MCP 名称列（双传输契约）。`discover_places` / `arrange_day` 为行程拆分工具（见 §9.2）。单个工具内部并行调用 AMAP+Google 属于**适配器扇出**。Tripadvisor 富化和 Open-Meteo 均在**服务端**处理。定时行程（`detail: "timed"`）在 `plan_itinerary` **内部**编排 geocode / search / weather — 仍然是一个公开 HTTP/MCP 工具。

共享输入：`providers[]`、`locale` 或 `locales[]`、`enrich.tripadvisor?`、`merge?`。核心层根据环境变量与能力矩阵校验 `providers[]`；**绝不**地理强制使用 AMAP（ADR-005）。

### `plan_itinerary` 详情模式

| `detail` | 行为 |
| --- | --- |
| `stops`（默认） | 按天重新分配调用方 `places[]` + 每日天气（MVP-2）。地点为空 → `errors.no_places_to_plan`。 |
| `timed` | 出发地可选。`days[].day_index` **从 1 开始**。`search_anchor` 是**城市**（来自自然语言/已知城市名），而非目的地地标。自动 `search_places` 使用景点允许列表 + 广场/商场/车站/景区拒绝列表（**无**未过滤回退）。当 locale 为 CN/HK/TW 或出发地/目的地/自然语言含有 CJK 字符时，组装的供应商查询使用中文。多天出发地+目的地：在插值走廊定位点处逐天搜索。为每天在范围内的每个时间段排列带时钟的访问 `blocks[]`。午餐取自访问间隙；**晚餐 18:00–20:00**；最后一次访问在 17:00 前结束时可选 `meal: "cafe"`。有营业时间数据时进行过滤。场所身份（`native_id` 或标准化名称）在整个行程中唯一，包括每个餐饮选项；候选不足时额外发起餐厅/咖啡馆查询；仍不足的时段予以省略（不回退到已使用过的场所）。`duration_min > 300` 的餐饮选项将被丢弃。目的地可选地对后续天数 + `legs_to_destination` 施加偏移。CN/HK/TW：若调用方列出了 AMAP，定时搜索**优先 AMAP** 再由 Google 补充（ADR-005：禁止注入 AMAP）。GCJ-02 `near` 坐标不做 GPS 转换。附近搜索半径为城市级别。模块：[`itinerary-timed.ts`](../src/core/itinerary-timed.ts)、[`itinerary-weather.ts`](../src/core/itinerary-weather.ts)、[`place-filters.ts`](../src/core/place-filters.ts)。**路线：** Google 和/或 AMAP `directions()` → `source: "directions"`；失败 → 启发式 + `errors.directions_unavailable` 并标注失败的供应商。AMAP 搜索失败时重试一次。 |

`planning_impact.severity`：根据 WMO 代码 + 高温（`temp_max_c ≥ 32`）得出 `fair` \| `caution` \| `adverse` \| `severe`。标签通过 `itinerary.weather.*` key 提供（ADR-014）。

网关：校验 → 并行扇出 → 标注 `sources[]` → 可选富化 → 可选合并 → 封装响应。

### 5.1 工具能力规格

每个工具返回结构化数据。本节定义每个工具的**完整字段契约** — 调用方可以期望得到什么、哪个供应商提供该字段，以及当供应商无法提供时的回退行为。

#### `search_restaurants` / `search_places`

| 字段 | 类型 | 供应商 | 回退 |
| --- | --- | --- | --- |
| `name` | string | AMAP / Google | 必填 — 缺失则跳过该卡片 |
| `address` | string | AMAP / Google | 必填 |
| `location` | `{ lat, lng, crs }` | AMAP (GCJ-02) / Google (WGS84) | 必填 |
| `rating` | number? | AMAP / Google | 不可用时省略 |
| `hours` | string[]? | AMAP (`opentime_today/week`) / Google (`regularOpeningHours`) | 不可用时省略 |
| `photos` | string[]? | Google (free tier) → Tripadvisor (`/locations/{id}/photos`) → AMAP (`biz_ext.photos`) | 见下方照片回退链。若无供应商返回图片，省略该字段（不返回空数组）。 |
| `types` | string[]? | AMAP / Google | 不可用时省略 |
| `price_level` | string? | Google / AMAP / Tripadvisor (enrich) | 见下方价格归一化。若无供应商返回价格数据，省略该字段。 |
| `price_per_person` | number? | AMAP (`biz_ext.cost`, 元) | 不可用时省略；仅 AMAP 提供数值型人均消费 |
| `provider` | string | System | 必填 — `AMAP` 或 `GOOGLE_MAPS` |
| `sources[]` | array | System | 必填 — 每个参与供应商对应一条记录 |

#### `suggest_places`

厂商 **autocomplete / 输入提示**（起点酒店前缀）。与 `search_places` 同 envelope（`PlaceCard[]`）；坐标可能缺失（NaN sentinel）— 调用方须 hydrate 或按名称+目的地过滤。

| 入参 | 说明 |
| --- | --- |
| `query` | 用户原文（去括号由调用方处理） |
| `address` / `city` | 目的地城名；AMAP 用 `citylimit=true` |
| `near` | 可选；Google `locationBias` 50km；AMAP tip `location` |
| `providers[]` | **省略** — ADR-052 策略选图 |

| 供应商 | 实现 |
| --- | --- |
| AMAP | `GET /v3/assistant/inputtips` |
| Google | `POST …/places:autocomplete`（Worker MCP 无此工具时返回空，由调用方 `search_places` 兜底） |

空提示 → `errors.empty_results`。**禁止**源码品牌/城市酒店表（ADR-042）。

**供应商组合策略**（MVP-3，取代 ADR-005 的仅调用方路由）：

智能体根据目的地和界面语言自动选择 provider 组合。Caller 仍可通过显式 `providers[]` 覆盖。

| 策略 | 条件（任一命中） | search providers | enrich |
| --- | --- | --- | --- |
| **策略1** | 目的地在中国大陆之外；或界面语言 EN/TW/HK | `GOOGLE_MAPS` | `TRIPADVISOR` (rating + photos fallback + reviews) |
| **策略2** | 目的地在中国大陆或香港 | `AMAP` | — |

两个策略可同时生效。示例：

| 目的地 | 语言 | 生效策略 | 实际 providers |
| --- | --- | --- | --- |
| 上海 | CN | 策略2 | AMAP |
| 上海 | EN | 策略1 + 策略2 | Google + AMAP + TripAdvisor enrich |
| 昆明 | EN | 策略1 + 策略2 | Google + AMAP + TripAdvisor enrich |
| 香港 | CN | 策略1 + 策略2 | Google + AMAP + TripAdvisor enrich |
| 香港 | HK | 策略1 + 策略2 | Google + AMAP + TripAdvisor enrich |
| 台湾 | CN | 策略1 | Google + TripAdvisor enrich |
| 台湾 | TW | 策略1 | Google + TripAdvisor enrich |
| 东京 | EN | 策略1 | Google + TripAdvisor enrich |
| 里斯本 | EN | 策略1 | Google + TripAdvisor enrich |

实现模块：`src/adapters/provider-resolver.ts` → `resolveProviderStrategy(destination, locale)` → `{ searchProviders[], enrichProviders[] }`。

**Photos 回退链：**

```
1. Google Places Photos (free tier, field mask `places.photos`)
   ↓ 不可用或需付费
2. Tripadvisor Terra `/locations/{id}/photos` (需先 nearby 匹配拿到 location_id)
   ↓ 不可用
3. AMAP `biz_ext.photos`
   ↓ 不可用
4. 省略 photos 字段（不返回空数组）
```

只有当 Google Photos 不可用或需要付费时，才回退到 Tripadvisor photos。AMAP photos 作为最后回退。

**Price 归一化：**

各 provider 返回不同格式的价格数据，统一为 `price_level` 枚举：

| `price_level` 值 | 含义 | Google `priceLevel` | AMAP `biz_ext.cost` (元) | TripAdvisor `price_level` |
| --- | --- | --- | --- | --- |
| `"FREE"` | 免费 | `PRICE_LEVEL_FREE` | — | — |
| `"$"` | 平价 | `PRICE_LEVEL_INEXPENSIVE` | < 50 | `$` |
| `"$$"` | 中等 | `PRICE_LEVEL_MODERATE` | 50–150 | `$$` |
| `"$$$"` | 较贵 | `PRICE_LEVEL_EXPENSIVE` | 150–300 | `$$$` |
| `"$$$$"` | 高档 | `PRICE_LEVEL_VERY_EXPENSIVE` | > 300 | `$$$$` |

- Google field mask 增加 `places.priceLevel`
- AMAP 请求增加 `show_fields=biz_ext` 或 `extensions=all`，解析 `biz_ext.cost`（人均元），按上表转换为 `price_level`；同时将原始数值保留在 `price_per_person` 字段
- Tripadvisor enrich 时如主 provider 无 `price_level`，使用 Terra 返回的 `price_level` 补充
- 多 provider 合并时：主 provider 的 `price_level` 优先；无数据则取 enrich provider 的值

**实时探测（2026-08-20，`PLACES_VENDOR_MODE=live`）：** Clerkenwell Google **11/20** 张卡片有 `price_level`；上海 AMAP **12/20** 张有 `price_level` + `price_per_person`；香港中环合并结果 **6/21** 张有 `price_level`，**5/21** 张有 `price_per_person`。参见 [`../knowledge/maps/price-level-live.md`](../knowledge/maps/price-level-live.md)。

#### `get_place_details`

只请求调用方给出的 `provider` + `native_id`；传入 UI `locale`（Google Details 必须带 `languageCode`）。不 fan-out 第二家供应商（[ADR-052](../adr/ADR-052-map-provider-routing.md) D9/D10）。

返回与搜索相同的字段，另加：

| 字段 | 类型 | 供应商 | 回退 |
| --- | --- | --- | --- |
| `reviews` | object[]? | Google / Tripadvisor (enrich) | 不可用时省略 |
| `website` | string? | Google | 不可用时省略 |
| `phone` | string? | AMAP / Google | 不可用时省略 |

#### `plan_itinerary` (detail: "timed")

**语言感知查询生成**（与 §5.2 QLP 对齐）：
- **Agent 自组关键词**的路径（timed 自动搜、`discover_places` / LLM Phase1）按 **provider 联动 QLP** 拼词 — 不是「仅 UI locale → 中/英」
- 关键词来自 [`search-keywords.ts`](../src/i18n/search-keywords.ts)；禁止写死英文 `attractions landmarks…` 之类硬编码句
- 模板：`"{city} {localized_keyword}"`（或 catalog 多词模板）
- 餐饮场景：午餐/晚餐/咖啡馆关键词按 **该次 job 的关键字 locale**（AMAP→CN；Google→EN 或 UI locale）区分

**行程中的场所照片**（MVP-3）：
- `blocks[]` 中每个场所在搜索供应商返回数据时包含 `photos` 字段
- 回退：省略 `photos`，绝不返回空数组

#### `geocode`

| 字段 | 类型 | 供应商 | 回退 |
| --- | --- | --- | --- |
| `location` | `{ lat, lng, crs }` | AMAP / Google | 必填 |
| `formatted_address` | string | AMAP / Google | 必填 |
| `place_id` | string? | Google | AMAP 时省略 |

#### `navigate`

| 字段 | 类型 | 供应商 | 回退 |
| --- | --- | --- | --- |
| `deeplinks` | object | AMAP / Google | 必填 — 不含密钥的 URL |
| `distance_m` | number? | AMAP / Google directions | 路线不可用时省略 |
| `duration_min` | number? | AMAP / Google directions | 路线不可用时省略 |

#### `visa_requirement`（MVP-11，ADR-044）

**HTTP：** `POST /v1/visa_requirement` — JSON 响应（非 NDJSON）。**MCP：** 同名工具。

| 输入 | 类型 | 说明 |
| --- | --- | --- |
| `passport` | string | ISO 3166-1 alpha-3（如 `CHN`） |
| `destination` | string | ISO 3166-1 alpha-3（如 `JPN`） |
| `locale` | `EN`\|`CN`\|`HK`\|`TW`? | 映射 Orizn `lang`（缺省 `EN`） |

| 输出字段 | 类型 | 说明 |
| --- | --- | --- |
| `requirement` | string | `visa_free` \| `visa_required` \| `e_visa` \| … |
| `visa_free_days` | number \| null | 免签天数 |
| `description` | string? | locale 化概述 |
| `documents` | string[]? | 所需材料 |
| `process` | string[]? | 申请/入境流程 |
| `processing_time` / `validity` / `max_stay` | string? | 审理时间 / 有效期 / 停留 |
| `extension` | object? | `{ possible, details? }` |
| `last_verified` | string \| null | 数据核验日期 |
| `source_url` | string \| null | 官方来源 URL |
| `unavailable_fields` | string[]? | 免费档 upgrade 占位字段名 |

错误 key：`errors.visa_invalid_country_code`、`errors.visa_unconfigured`、`errors.visa_quota_exceeded`。

**智能体指令示例（管理端 §12.5）：**

```http
POST /v1/visa_requirement
Authorization: Bearer <caller-key>
Content-Type: application/json

{ "passport": "CHN", "destination": "JPN", "locale": "CN" }
```

MCP：`visa_requirement` 同名；description 须含字面量 `places-agent`。

### 5.2 语言路由与查询组装（MVP-4 + itinerary QLP）

三个协同模块 — **provider 选择**、**语言检测（UI/prompt）**、**query 组装（地图搜词）** — 共同决定「agent 自组关键词」时如何搜。均为规则引擎，不调 LLM。

**与 prompt-assembler 的区别（勿混用）：**

| 模块 | 服务对象 | 输出 |
| --- | --- | --- |
| [`prompt-assembler.ts`](../src/agent/prompt-assembler.ts) | **LLM** system / 场景 prompt | 说明文案 |
| [`query-assembler.ts`](../src/core/query-assembler.ts) | **AMAP / Google** 搜索 API | `{ providers, query }[]` jobs |
| [`language-router.ts`](../src/agent/language-router.ts) | UI / prompt locale 检测 | `LanguageContext`（`searchLocale` / `promptLocale`） |

**QLP 适用范围（锁定）：**

| 路径 | 是否走 QLP |
| --- | --- |
| `discover_places`、LLM Phase1 `searchCandidates`、timed `plan_itinerary` 自动搜景点/餐厅 | **是** — agent 自组关键词 |
| 公开 `search_restaurants` / `search_places`（调用方或 chat 模型自带 `query`） | **否** — 保留调用方原文；仅 `applyProviderStrategy` 选地图 |

```typescript
// src/agent/language-router.ts — UI / prompt（不单独决定地图搜词语言）
interface LanguageContext {
  detectedLanguage: "zh" | "en" | string;
  searchLocale: Locale;   // catalog lookup for UI-facing keyword needs
  promptLocale: Locale;   // system prompt selection
}

// src/adapters/provider-resolver.ts — ADR-026 / ADR-030
interface ProviderStrategy {
  searchProviders: ProviderId[];  // GOOGLE_MAPS, AMAP — order matters
  enrichProviders: ProviderId[];  // TRIPADVISOR
}

// src/core/query-assembler.ts — itinerary-composed search only
type SearchJob = { providers: string[]; query: string };
// assembleAttractionSearchJobs / assembleRestaurantSearchJobs → SearchJob[]
```

#### 5.2.1 语言检测（UI / prompt）

1. 显式 `locale` 参数 → 直接使用  
2. 输入中 CJK 字符占比 >30% → `zh` / `CN`  
3. 回退 → `en` / `EN`  

用于 prompt 与 QLP-G 的「第二趟 UI 语言」；**不能**单独决定 AMAP 搜词（见 QLP-A）。

#### 5.2.2 Provider 策略

按 §5.1 / ADR-030：目的地区域 → `{ searchProviders, enrichProviders }`（大陆 AMAP；港 AMAP+Google；其他 Google）。Caller 显式 `providers[]` **始终覆盖**自动策略。

`search_restaurants` / `search_places` / discover 在省略 `providers[]` 时均经 `applyProviderStrategy` / `resolveProviderStrategy`。

#### 5.2.3 Query Language Policy (QLP)

**按「这次 job 打哪家地图」拼关键字**，不是「界面是 CN 就中文、EN 就英文」。

| 策略 | Query 语言 | 适用条件 |
| --- | --- | --- |
| **QLP-A** (AMAP) | **纯简体 CN**（catalog `CN`） | `providers` 含 `AMAP` 的 job |
| **QLP-G** (Google) | EN；若 UI ≠ EN 再并行一趟 UI locale | `providers` 含 `GOOGLE_MAPS` 的 job |

**细则：**

- **QLP-A：** 无论 UI 是 EN/CN/HK/TW，AMAP job **只用简体**；禁止英文景点句打高德（哈尔滨实测英文 attractions → 0，中文「景点」→ 有结果）。
- **QLP-G：** UI=EN → 仅英文；UI≠EN → EN + UI locale **并行**，合并去重。
- **双 provider**（如 where2play 传 `[AMAP, GOOGLE_MAPS]`）：**拆成多 job**，禁止一个英文 query 同时 fan-out 两家。
- **延迟封顶：** AMAP 景点模板 ≤2；Google 景点 1～2 job；餐厅每 provider 通常 1（+ UI 双语时 +1）。

**where2play 主路径：**

```
POST /v1/discover_places
  { city, bounds, origin?, locale, numDays?, providers? }
  → resolve providers（caller 或 auto）
  → assembleAttractionSearchJobs + assembleRestaurantSearchJobs
  → parallel searchPlaces / searchRestaurants per job
  → merge by name → candidates
POST /v1/arrange_day { candidates, dayIndex, … }
```

BFF 不组地图关键词；关键词政策在 places-agent。

**关键词映射表** (`src/i18n/search-keywords.ts`)：

| EN | CN | HK | TW |
| --- | --- | --- | --- |
| Japanese restaurant | 日料 | 日本料理 | 日本料理 |
| hotpot | 火锅 | 火鍋 | 火鍋 |
| barbecue / BBQ | 烧烤 | 燒烤 | 燒烤 |
| brunch | 早午餐 | 早午餐 | 早午餐 |
| fine dining | 精致餐厅 | 高級餐廳 | 精緻餐廳 |
| cafe / tea house | 咖啡馆 / 茶馆 | 咖啡店 / 茶館 | 咖啡廳 / 茶館 |
| museum | 博物馆 | 博物館 | 博物館 |
| night market | 夜市 | 夜市 | 夜市 |
| budget | 平价 | 平價 | 平價 |
| premium | 高档 | 高檔 | 高檔 |

表不穷举；未命中保持原语言。景点另有 `viewpoint` / `park` / `historic` 等键（CN「景点」等）。

**实例（itinerary / discover）：**

| 场景 | UI | Providers | 实际搜词 jobs |
| --- | --- | --- | --- |
| 哈尔滨发现 | CN | AMAP+Google（caller） | AMAP: CN 景点模板；Google: EN（+ CN） |
| 哈尔滨发现 | EN | AMAP+Google | AMAP: **仍 CN**；Google: EN only |
| 东京发现 | EN | Google | Google: EN |
| timed 上海 | EN UI + 城市含 CJK | AMAP wave | CN catalog（不得因 UI=EN 用英文 attractions） |

**性能：** 同 provider 多 job / 双语 Google 用 `Promise.all`；合并按 `name`（discover）或 timed 既有 `native_id` / 使用集合。

**双路由区域（香港）跨供应商同地点：** Google + AMAP 可能各返回同一物理地点（不同 `native_id`）。stops pool 按 `(provider, native_id)` 各存一行（[ADR-056](../adr/ADR-056-registry-backfill-semantics.md)）；去重不在写库/读池硬合并，由环内 LLM 取点时自行判断 — 见 §9.2 与 [ADR-058](../adr/ADR-058-cross-provider-duplicate-llm-judges.md)。

### 5.3 提示组装

> **实现与契约见 §9.1（MVP-6）。** 本节不再单独维护「MVP-7」草稿。

Chat / tool 的 system prompt 由 [`prompt-assembler.ts`](../src/agent/prompt-assembler.ts) 按 `locale` + `intent` 拼接；**地图搜词**见 §5.2.3 query-assembler，二者分离。

---

## 6. 适配器与地点卡片

| 适配器 | Id | 说明 |
| --- | --- | --- |
| AMAP Web 服务 | `AMAP` | `PLACES_VENDOR_MODE=live` 时使用实时模式：[`config.ts`](../src/adapters/amap/config.ts)、[`direct.ts`](../src/adapters/amap/direct.ts)、[`card-mapper.ts`](../src/adapters/amap/card-mapper.ts)（将 `opentime_today` / `opentime_week` 映射至 `PlaceCard.hours`）、[`keywords.ts`](../src/adapters/amap/keywords.ts)、[`directions.ts`](../src/adapters/amap/directions.ts)（`/v3/direction/walking|driving|transit/integrated`）、[`live.ts`](../src/adapters/amap/live.ts)。`lng,lat`；GCJ-02；通过 `/v3/assistant/coordinate/convert`（`coordsys=gps`）转换 WGS `near` 坐标；餐饮 `types=050000`；有 `address` 无 `near` → 先地理编码再以 `/v5/place/around` 半径 1000 搜索。非实时模式时使用 Fixture：[`fixture.ts`](../src/adapters/amap/fixture.ts)。**无** `weatherInfo`。无 Worker 回退。 |
| Google direct REST | `GOOGLE_MAPS` | Places New / Geocoding / Routes；WGS-84；`languageCode` 来自 locale 映射表。模块：[`src/adapters/google/direct.ts`](../src/adapters/google/direct.ts) |
| Google Worker MCP | 同 `GOOGLE_MAPS` | 直连出站失败后使用。`GMAPS_MCP_*`。首先调用 `tools/list`。模块：[`src/adapters/google/mcp-client.ts`](../src/adapters/google/mcp-client.ts)。组合模块：[`src/adapters/google/live.ts`](../src/adapters/google/live.ts)。**开发测试：** `GOOGLE_DIRECT_FORCE_FAIL=1`（生产环境拒绝使用）。 |
| Tripadvisor Terra | `TRIPADVISOR` | **仅用于富化**（ADR-007，ADR-020）：评分、评论、**照片回退**（MVP-3）。`PLACES_VENDOR_MODE=live` 时使用实时模式：[`config.ts`](../src/adapters/tripadvisor/config.ts)、[`direct.ts`](../src/adapters/tripadvisor/direct.ts)、[`match.ts`](../src/adapters/tripadvisor/match.ts)、[`card-mapper.ts`](../src/adapters/tripadvisor/card-mapper.ts)、[`live.ts`](../src/adapters/tripadvisor/live.ts)。`GET /locations/nearby` 携带 `lat`+`lon`+`radius=1`+`unit=KM`；请求头 `X-API-Key`；附近搜索 URL 中不传 `location_id` 或 Google/AMAP 原生 id。**照片：** `GET /locations/{id}/photos` — 仅在 Google Photos 不可用或需付费时调用；`location_id` 来自附近搜索步骤。非实时模式时使用 Fixture：[`fixture.ts`](../src/adapters/tripadvisor/fixture.ts)。 |
| Open-Meteo | `OPEN_METEO` | **不**出现在 `providers[]` 中。`PLACES_VENDOR_MODE=live` 时使用实时模式：[`config.ts`](../src/adapters/open-meteo/config.ts)、[`direct.ts`](../src/adapters/open-meteo/direct.ts)、[`live.ts`](../src/adapters/open-meteo/live.ts)。`GET /forecast` 携带 `latitude`+`longitude`+`daily=weather_code,temperature_2m_max,temperature_2m_min`+`timezone=auto`；客户主机上可选 `apikey`。非实时模式时使用 Fixture：[`fixture.ts`](../src/adapters/open-meteo/fixture.ts)。保留 `weather_code` + 数值；本地化 `weather.wmo.{code}`。 |
| Orizn Visa | `ORIZN_VISA` | **不**出现在 `providers[]` 中（与 Open-Meteo 同类辅助数据源）。`PLACES_VENDOR_MODE=live` 时 REST 直连 `GET /api/v1/visa`（`x-api-key`）；**不** spawn `orizn-visa-mcp` 子进程（远程 MCP 端点即使带 Key 也仅暴露 `quick_visa_check`，不足材料/流程级答案 — 见 [ADR-044](../adr/ADR-044-orizn-visa-rest-adapter.md)）。模块：`src/adapters/orizn/{config,direct,fixture,live}.ts`。进程内 `(passport, destination, lang)` TTL 缓存（默认 24h，`ORIZN_CACHE_TTL_H`）。fixture 样本：CHN→JPN、CHN→SGP 等。 |

| 字段 | 规则 |
| --- | --- |
| `provider` | 主供应商 id |
| `sources[]` | `{ provider, native_id, logo_url?, deeplinks }` — native id 仅属于**该**供应商 |
| `primary_provider` | `merge: true` 时使用 |
| `location` | 每个来源对应 `{ lat, lng, crs: "WGS84" \| "GCJ-02" }`；不得在同一定位点混用坐标系 |
| `name` / `address` | 第 B 层：供应商字符串；不使用术语表替换 |
| Deeplinks | 不含密钥 |
| Photos | URL 中不含密钥；必要时通过 BFF 代理至 `/v1`；绝不使用 `NEXT_PUBLIC_` |

---

## 7. HTTP 响应封装

所有 `/v1` JSON 响应体（含健康检查）：

```ts
{
  agent: "places-agent", // literal, never localized
  ok: boolean,
  data?: unknown,
  outcome?: { key: string; locales?: Partial<Record<Locale, string>> },
  skipped?: { provider: string; reason_key: string }[],
  locale?: Locale,
  locales?: Locale[]
}
```

| Key | 典型 HTTP 状态码 |
| --- | --- |
| `errors.caller_unauthorized` | 401 |
| `errors.place_not_found` | 404 |
| `errors.empty_results` | 200 + 空列表 |
| `errors.provider_failed` / `unconfigured` / `capability_unsupported` | 200 + `skipped[]` |
| `errors.upload_unsupported` / `upload_too_large` | 400 |

始终输出 **key**。按请求的 locale 解析目录文本；回退顺序：请求 locale → `EN` → 原始 key。不允许 HK↔TW 互相回退。

---

## 8. MCP

| 项 | 契约 |
| --- | --- |
| 传输 | Streamable HTTP `POST/GET /mcp` |
| ChatBox | 在**同一** `McpServer` 上的 `GET /sse` + `POST /messages` |
| 认证 | initialize **之前**验证调用方 Bearer |
| `serverInfo.name` | `"places-agent"` |
| 工具名称 | 不加前缀 |
| 工具描述 | 必须包含字面量 `places-agent`，以便 ChatBox/Cursor 宿主模型能够优先选用这些工具，而非通用搜索或供应商地图 MCP |
| 模式 | Zod，与 HTTP 共享 |
| 结果 | 与 `/v1` 的 `data` + outcome key 含义相同 |

`registerTools(server)` → 仅使用 `core.*`。

**第三方宿主（Cursor / ChatBox）配置、时序图、宿主契约：** 见文首 [第三方工具调用](#core-mcp-hosts)。运维细节：[`mcp-client-integration.md`](../knowledge/agent/mcp-client-integration.md)。

---

## 9. 智能体 LLM（Qwen，ADR-047）

- 使用 `openai` SDK，`baseURL` = `QWEN_BASE_URL`（compatible-mode），而非 `api.openai.com`。
- 模型：`QWEN_CHAT_MODEL`（默认 `qwen-plus`）。`QWEN_API_KEY` 为空时回退 `OPENAI_*`。
- 上限：最大迭代次数 + 出站 HTTP 超时约 25s；界面软提示约 10s（统一延迟契约）。

**版本（git 即注册表）：**

| 制品 | 示例 id | 位置 |
| --- | --- | --- |
| 系统提示 | `chat.v1` | `prompts/chat/v1.md` |
| 旅行术语表 | `travel.v1` | `prompts/glossaries/travel.v1.json` |
| 目录包 | `catalogs.v1` | `messages/*.json` |

环境变量：`PROMPT_ID`、`GLOSSARY_ID`（`EN`/`CN` 时为 null）、`CATALOG_PACK`。回滚 = 锁定 id。发布后不得原地修改 `v1`。

### 9.1 Prompt 组装器 (MVP-6)

**模式：** 基础模板 + 场景片段拼接。按 locale 选 base prompt，按 intent 追加 overlay。

```
prompts/
  base.en.md                    — 角色定义 + 通用规则（英文；由 chat/v1 迁移）
  base.zh.md                    — 角色定义 + 通用规则（中文）
  overlays/
    meal-search.md               — 餐厅搜索场景指引
    place-search.md              — 景点搜索场景指引
    itinerary-planner.md         — 行程规划 prompt（含自查指令 + JSON 期望）
```

**未落地独立文件：** `budget` / `time-of-day` 作为字符串常量**内联**于 [`prompt-assembler.ts`](../src/agent/prompt-assembler.ts)（与 Claude Code Plan 一致）。无 `itinerary-reviewer.md`（单 LLM 自查，无第二 Reviewer 调用）。

```typescript
// src/agent/prompt-assembler.ts
interface PromptContext {
  locale: Locale;
  intent: "meal" | "place" | "itinerary" | "chat";
  budget?: "budget" | "premium";
  timeOfDay?: "morning" | "afternoon" | "evening";
  glossary?: string;
}

function assembleSystemPrompt(ctx: PromptContext): string;
```

拼接顺序：`base.{en|zh}.md` → `overlays/{intent}.md`（chat 可无 overlay）→ 内联 budget 提示（可选）→ 内联 time-of-day 提示（可选）→ glossary（HK/TW 时）。

### 9.2 行程规划：MCP 工具拆分 + Token 优化 (MVP-6)

**性能与 MCP 路由：** 见 [`performance.md`](./performance.md) **v2.4** + [ADR-036](../adr/ADR-036-where2play-assistant-quanzil.md) + [ADR-037](../adr/ADR-037-where2play-plan-l2-quanzil.md) + [ADR-040](../adr/ADR-040-plan-itinerary-align-split-tools.md) — 形成行程 **必须 LLM**；**Mode H**（`execution=host`）已交付（Feature **35**）；**MCP 缺省 `execution=agent`**（ADR-040 D4'：不要求改客户端 system prompt）；**2play 初排 L2 + 助手 = 本应用 OPENAI_CN**（**Paused as-built**；**Target / T1：** [ADR-050](../adr/ADR-050-where2play-no-product-llm.md) Accepted + 本节「真智能体」/「MVP-T1 as-built」）；禁叠跑。

**MCP 工具拆分：** 将行程规划拆为可逐步返回的工具；`plan_itinerary` 仍为一站式 HTTP/MCP 入口。

| 工具 | 职责 | MCP | HTTP |
|------|------|-----|------|
| `discover_places` | 搜景点+餐厅+天气，返回候选列表（L1，无 LLM） | ✅ | ✅ `/v1/discover_places` |
| `arrange_day` | 从候选中为第 N 天安排路线；`execution=agent`（默认）跑服务端 LLM，或 `execution=host` 仅返回 prompt | ✅ | ✅ `/v1/arrange_day` |
| `plan_itinerary` | 一次返回完整行程（内部可走 LLM 或 legacy） | ✅ | ✅ `/v1/plan_itinerary` |

**Mode H（Feature 35）：** `arrange_day` + `execution: "host"` → `{ execution: "host", system_prompt, user_prompt, output_contract, candidates_slim }`；**本请求不调 OpenAI**；共享 `buildSchedulePrompt`（MCP 与 HTTP）。宿主（ChatBox / Cursor / 2play `plan-11`）用自有模型执行。缺省或 `execution: "agent"` → 服务端 OPENAI_CN 结构化排程（既有行为）。

**对话默认（ADR-043）：**
- `discover_places`（缺字段 → 单条 `intake`）→ `arrange_day`（**强制 agent**）→ 按 `next_action` 先上屏当日 → `presented_previous_day=true` 再下一天  
- **HTTP Mode H：** 仅 2play / 显式 host；MCP 忽略 host  
- **一站式整包：** `plan_itinerary` / `trip_plan` / `trips`  
- **一站式：** `plan_itinerary`（内部搜索 + LLM/legacy）  
- **2play 主路径（ADR-037 as-built）：** `discover_places` only；L2 在 BFF OPENAI_CN（不默认 `execution=agent`）。**Target：** [ADR-050](../adr/ADR-050-where2play-no-product-llm.md) — 零产品 LLM，改走 `plan_trip`

**HTTP progressive（ADR-032 #5，where2play L1）：** 当请求头 `Accept: application/x-ndjson` 时：

| 端点 | 流事件（一行一 JSON） | 结束 |
| --- | --- | --- |
| `POST /v1/discover_places` | `{type:"candidate", kind:"place"\|"restaurant", card}` 每 POI | `{type:"discover_done", counts}`；无 Accept 时仍返回批量 JSON |
| `POST /v1/arrange_day`（`execution=agent`） | Zod OK 后 `{type:"place", dayIndex, block}` 按序每块 | `{type:"day_done"}`；无 Accept 时仍返回批量 JSON |
| `POST /v1/arrange_day`（`execution=host`） | 无 LLM 流；单次 JSON handoff | `{ execution, system_prompt, user_prompt, … }` |

- `discover_places`：`numDays` 传入真实 N（不得硬编码 1）；`city` = 目的地字符串；L1 = 通用热门模板 query + Google **RELEVANCE**（ADR-043；**禁止 POPULARITY**）；**无城市 CATALOG**（ADR-042/D9：源码禁任何城市 POI 知识，CATALOG 已清空）；must-see 由 LLM 从候选池推断（`discover-must-see-llm.ts`）。  
- MCP `arrange_day`：**强制 agent**（忽略 `execution=host`）；返回 `start_time` / `legs_to_here` / `next_action`；软闸 `presented_previous_day`（ADR-043）。  
- HTTP `arrange_day`：仍可 `execution=host`（2play Mode H）。
- `arrange_day`：可选 `exclude_names: string[]`；硬必去 Feature **36**；真交通 enrichment Feature **37**（`legs_to_here`）。
- **MCP 工具保持 request/response**（无 NDJSON）；session 见 Feature **38**。

**Token 优化：**

| 参数 | 旧值 | 新值 | 效果 |
|------|------|------|------|
| 候选数 | 15/type | **8/type** | user message -50% |
| max_completion_tokens | 4096 | **arrange 1280 / multi-day 2048** | 单日更短；多日仍 2048 |
| 候选描述 | name+type+rating+lat/lng+hours+price | **name+type+rating+lat/lng** | -30% |
| LLM 超时 | 无限制 / SDK timeout | **AbortSignal 硬中断 45s**（`LLM_ARRANGE_TIMEOUT_MS` / `LLM_ITINERARY_TIMEOUT_MS`）；校验失败才重试一次；超时不重试 | 避免 OPENAI_CN 挂死；用户单次最多 ~45s |

**行程配图：** Phase 4 格式化时，用 block.name 匹配候选的 `photos` 字段（来自 MVP-3b），挂回每个 block。封面图 = Day 1 第一个 attraction 的第一张 photo。零额外 API 调用。

**架构流程（单 LLM + 自查 + Zod）：**

```
discover_places(city, bounds, locale, providers?):
  providers = caller providers[] OR resolveProviderStrategy(city)   // ADR-030
  // ADR-042/043 D9: no city CATALOG in source. Generic hot templates (e.g. "西安 景点")
  // + Google rankPreference=RELEVANCE (never POPULARITY). Must-see identification
  // comes from LLM inference over the pool (discover-must-see-llm.ts), not a seed encyclopedia.
  jobs = discover query-assembler: generic templates + QLP           // §5.2.3 + ADR-038/042
  parallel searchPlaces / searchRestaurants per job
  merge by name → filterAttractionPlaces / filterDiningPlaces
       → deny fragments (票/直通车/敌楼/「主名-」后缀)
       → dedupeByCluster → ensureMustSeeDiversity
  rank: rating + inferred must-see (LLM) — no per-city hardcoded boost
  → top 8×min(numDays,3) per pool
  → { candidates, weather? }
  // L1: no LLM for candidate search. Pool head must be diverse primaries, not wall-fragment spam.
```

arrange_day(candidates, day_index, origin, destination, pace, budget, locale, execution?):
  if execution == "host":
    return buildSchedulePrompt(...)   // no OpenAI; Feature 35
  // ADR-043 D9: deterministic injection removed. LLM 漏排 must_include focus →
  // 硬失败重试一次（callItineraryLlmWithValidationRetry）；theme 门控 focus：
  // 仅当 day_theme 命中 missing token 才强制 focus，否则 token 留待后续 themed 日（末日门仍保证覆盖）。
  LLM 规划 + 自查（temperature 0.35，max_tokens=1280，AbortSignal 45s）
  Zod 校验 → 失败重试一次 → 超时/网络不重试 → 仍失败 → fallback 旧代码
  enrichArrangeTransit(...)           // Feature 37 legs_to_here（可降级）
  匹配候选 photos → 挂回 blocks
  → { day: { blocks, from_origin?, to_destination? } }

plan_itinerary(input):
  // LLM mode: Phase1 = same discover searchCandidatePools; or legacy timed with QLP-aware queries
  …
```

**行程优化模块（MVP-8 / ADR-040/043 D9）：**

| 模块 | 职责 |
| --- | --- |
| [`trip-intake.ts`](../src/core/trip-intake.ts) | MCP/HTTP arrange 边界收集 + intake 门（need_input）+ host_instructions RULE |
| [`must-include-coverage.ts`](../src/core/must-include-coverage.ts) | `must_include` 覆盖追踪、sticky covered、theme 门控 focus token 选择 |
| [`discover-must-see-llm.ts`](../src/core/discover-must-see-llm.ts) | LLM 从候选池推断公认 must-see（prompt 无城市名，替代硬编码 CATALOG） |
| [`discover-dedupe.ts`](../src/core/discover-dedupe.ts) | 地标 cluster 去重 + 池头多样性（无城市专属正则） |
| [`query-assembler.ts`](../src/core/query-assembler.ts) | discover/LLM Phase1/timed 的地图搜词 jobs（通用模板，无城市种子） |
| [`enrich-arrange-transit.ts`](../src/core/enrich-arrange-transit.ts) | Feature 37：arrange blocks 挂 `legs_to_here`/`from_origin`/`to_destination`，失败降级 heuristic + `transit_outcome` |
| [`arrange-present-gate.ts`](../src/mcp/arrange-present-gate.ts) | MCP 顺序展示软闸（`presented_previous_day`/`ack_day_index`）+ 续排 host_instructions |
| [`http-transport.ts`](../src/mcp/http-transport.ts) | Feature 38：SSE/Streamable 路由 + session 生命周期（缺/过期可恢复） |
| [`tests/no-city-hardcode-guard.test.ts`](../tests/no-city-hardcode-guard.test.ts) | 守卫：源码禁任何城市 POI 知识（ADR-042 原则钉成 CI 闸） |

**跨供应商同地点去重（[ADR-058](../adr/ADR-058-cross-provider-duplicate-llm-judges.md)）：**

双路由区域（如香港 Google + AMAP）下，同一物理地点可在 stops pool 中以不同 `(provider, native_id)` 存为独立行（[ADR-056](../adr/ADR-056-registry-backfill-semantics.md) D1）。读侧 `mergeRegistryPlaces` 仅按归一化名称软去重；名称漂移时候选列表仍可能含重复。

> **2026-09-11 amendment（ADR-067）：** T3++ 起 `mergeRegistryPlaces` 在 plan_trip 发现/骨架路径**停用** —— registry 退化为 cache-only，不再 merge 整城库进 trip 候选池。Trip candidates 只来自 LLM 提名 + grounding 的本 trip 名单。下段「池合并」描述适用于 T3 as-built 及非 plan_trip 读路径。

- **LLM 取点须自己判断：** 环内模型在 `search_places` 候选 + 池合并结果上，自行识别跨供应商同地点（名称 / 坐标 / 描述），并避免同日或跨日重复选取。骨架 / fill prompt **须含硬约束**「不得重复同一物理地点」。
- **代码不替模型去重：** 不在写库合并、不在读池硬聚类。事实闸（eligible、Directions、`(provider, native_id)` 唯一）仍由代码执行；「是否同一物理地点」是判断，不是事实闸。
- **已知残留：** 跨日 `native_id` 唯一校验（M05）不捕获跨供应商同地点（`native_id` 不同）；依赖 prompt 约束。若生产仍选重，另开故事评估可选读侧坐标聚类 — 不在本设计默写。

**新旧切换：**

| | 设计意图 | **当前实现**（`src/core/itinerary.ts`） |
|--|----------|----------------------------------------|
| 环境变量 | `ITINERARY_MODE=llm` \| `legacy` | 同名 |
| 默认值 | **`llm`** | **`llm`**（`process.env.ITINERARY_MODE ?? "llm"`） |
| 生产启用 LLM | 默认即 LLM | 旧路径测试须显式 `ITINERARY_MODE=legacy` |

旧代码路径保留不删。

**搜索范围：** 有城市名 → 5km 半径；无城市名 → `errors.location_too_broad`。

**出发地/返回地交通（每日酒店可选）：**  
- 聊天应**询问**每日酒店/地标起点，但**非硬门禁**（用户可不提供）。  
- **有** origin（名称或坐标）→ 含 `from_origin`（酒店→首站）与 `to_destination`（末站→回程）；站间必须有 `legs_to_here`。  
- **无** origin → **省略** `from_origin` / `to_destination`；行程自第一个 block 起、至最后一个 block 止；**站间仍须** `legs_to_here`（游中交通时间）。首 block `start_time` ≥ 10:00。  

**MCP 固定行程表（8 行，每次相同）：** 城市、开始日、天数、可选酒店、节奏（轻松/适中/紧凑，默认适中）、消费 `spend_level` 1 节约 / 2 适中 / 3 宽松（默认 2）、兴趣（可选）、必去/一日游地名。禁止随机少问。  

**必去覆盖闸（ADR-043 D7 + D9 精简；HTTP = MCP）：**  
- `preferences.must_include` 每次带回。  
- 共用 `arrangeDay`：对仍 missing 的 token **一次自动补搜一个**（theme 对齐优先，否则名单顺序）→ geocode 锚点 → search 合并进候选 → prompt HARD MUST SCHEDULE（可与城内点混排）。  
- **D9 精简（删确定性注入）：** LLM 漏排 focus token → **硬失败重试一次**（不再服务端造低质块注入）。  
- **theme 门控 focus：** 仅当本日 `day_theme` 命中某 missing token 才对该 token 强制 focus；无 theme 或 theme 不匹配 → 不强制 focus，token 留待后续 themed 日（末日门仍保证覆盖）。避免 day-trip 小镇被无 theme 的早期日抢排成半天。  
- 响应字段 `must_include_coverage: { must_include, covered, missing }`（HTTP envelope 与 MCP 同结构）。  
- 末日若仍有 missing → MCP `next_action: present_day_then_cover_must_include`，禁止总览。  

**空候选自动 discover（ADR-043 D8；HTTP = MCP）：**  
- 硬必填仅 city + 开始日 + 天数；调用方候选池可选。  
- `exclude_names` 后若 **`places` 为空**且 `city` 已给 → `arrangeDay` 内调 `discoverPlaces` 填景点（餐厅侧若亦空则一并填），再进 D7 / LLM。仅餐厅空、景点已有时不 live discover。  
- 无 city 且池空 → 清晰失败；失败文案禁止诱导宿主 invent POI。  
- 末日 host_instructions：Day 卡与总览各写一次后 STOP；日卡仅列工具返回的 `blocks[]`。  

**日卡版式：** 多行块（`### HH:MM–HH:MM｜店名` + 说明 + 前往 + 路线/地点链接）；禁止单行 `|` 压缩。  

**节奏与「排满」收工（默认 `medium`）：**  
| pace | 上限 blocks/日 | 收工期望 | 不满（须重试/补排） |
| --- | --- | --- | --- |
| `relaxed` | ≤4 | 末块结束 ≥ **17:00** | 末块结束早于 **16:00** |
| `medium`（默认） | ≤5 | 须含 **dinner**；末块结束约在晚餐结束（目标 **~20:00**，窗 18:00–20:30） | 无晚餐，或末块结束早于 **16:00**，或适中日在 **19:00** 前收工且无晚餐 |
| `tight` | ≤6 | 须含 **dinner**；末块结束 ≥ **19:30** | 同 medium 的不满底线，且过稀 |

**LLM 输出 schema（per day，Zod 校验）：**

```json
{
  "day_index": 1, "date": "2026-08-25",
  "from_origin": { "transport": "metro", "duration_min": 25, "depart_time": "09:30" },
  "blocks": [{
    "name": "精确匹配候选 name",
    "type": "attraction | lunch | dinner | cafe",
    "start_time": "10:00",
    "duration_min": 90,
    "reason": "推荐理由",
    "alternatives": [{ "name": "...", "reason": "..." }]
  }],
  "to_destination": { "transport": "taxi", "duration_min": 40, "arrive_time": "18:30" }
}
```

**边界条件：** origin ≠ destination → 搜索锚点逐天偏移；候选不足 → prompt 说明；Zod 2 次失败 → fallback + `outcomeKey`。

---

## 10. 数据（[ADR-025](../adr/ADR-025-places-agent-postgres-prisma.md)）

PostgreSQL + Prisma。本地 `DATABASE_URL=postgresql://places_agent:places_agent@localhost:5435/places_agent`（或 `:5436`）。生产环境使用阿里云专用数据库 `places_agent`，地址 `101.132.156.250:5432`。不使用挂载卷上的 SQLite（ADR-015 已废止）。不共享 `what2eat` 数据库。

| 实体 | 字段 |
| --- | --- |
| `AdminUser` | `id`、`username` 唯一、`email` 唯一、`passwordHash`（设置前为空）、邀请/重置 token **哈希值** + 过期时间 |
| `CallerApiKey` | `id`、`name`、`description`、`keyHash` 唯一、`prefix`、可选 `secret`（明文，供管理后台列表 Copy；迁移前行为 `null`）、`status` `ACTIVE`\|`REVOKED`、`lastUsedAt` |
| `Trip`（MVP-16 / ADR-046） | `id`、`revision`、`status`、`callerKey`、`locale`、`expiresAt`、分区 JSON（constraints / candidates / skeleton / cursor / filled / artifacts）；进程内内存热副本，见 §21 |
| Session | **封装 Cookie**，非数据表。后续可选：用户上的 `sessionVersion` 字段用于全部吊销 |

种子数据：用户名 `admin`，邮箱 `me@ethanhuang.com`。**不**将密码内置到镜像中。空哈希 → `/set-password` 或 Resend 重置。

密码：使用 `node:crypto` scrypt 算法。调用方密钥：`pa_` + 32 字节随机数；存储 SHA-256 `keyHash` 用于 Bearer 鉴权，并持久化明文 `secret` 供管理后台列表 Copy（[ADR-034](../adr/ADR-034-caller-api-key-secret-at-rest.md)）。创建/重新生成响应仍返回 `secret`；`GET /api/admin/api-keys`（仅会话）亦返回 `secret`（旧行可为 `null`）。

---

## 11. 认证

| 渠道 | 凭证 | 无效场景 |
| --- | --- | --- |
| 运营者后台 + `/api/admin` | 会话 Cookie | `/v1`、`/mcp`、`/sse` |
| HTTP 工具接口 + MCP | `Authorization: Bearer` | 管理页面 / `/api/admin` |

Cookie：`HttpOnly`、`Secure`（生产环境）、`SameSite=Lax`、`Path=/`。HTTPS 有保证时优先使用 `__Host-places_agent_session`。载荷 `{ userId, username }` 以 `SESSION_SECRET` 封装。

CSRF（仅 Cookie 写操作）：`SameSite=Lax` **加上** `Origin` / `Referer` 必须匹配本主机。Bearer 接入面没有 CSRF Cookie 攻击风险；不要从运营者浏览器发送调用方密钥。

调用方密钥：对 Bearer 取哈希，查询 `ACTIVE` 状态。缺失/未知/已吊销/地图供应商密钥作为 Bearer → `errors.caller_unauthorized`。使用时序安全比较。

---

## 12. 管理后台

部署在 **`places.agent-mate.ai`** 的运营者管理网页。功能 14–19 的像素与交互契约。可点击原型：[`ui-mockup/`](./ui-mockup/)。locale Cookie 为 `places_locale` — **不使用** `[locale]` 路径段。

| URL | 功能编号 | 原型文件 | 认证 |
| --- | --- | --- | --- |
| `/` | 14 | `01-home.html` | 公开 |
| `/login` | 15 | `02-login.html` | 公开；已登录 → `/admin/api-keys` |
| `/reset-password` | 15 US3 | `03-reset.html` | 公开 |
| `/set-password` | 15 US5 | `04-set-password.html` | 仅限重置 token 或空密码会话 |
| `/accept-invite` | 15 US4 | （已实现；原型待定） | 邀请 token；用户资料与密码引导 |
| `/instructions` | 18 | `05` / `11` | 公开或会话；内容相同；字面量 `places-agent` |
| `/admin` | 16 | — | 会话；**重定向** → `/admin/api-keys` |
| `/admin/api-keys` | 16 + 17 (US5 bulk delete) | `06-keys.html` | 会话；**登录后落地页** |
| `/admin/api-keys/new` | 17 US1 | `07-key-new.html` | 会话 |
| `/admin/api-keys/[id]` | 17 US2–4 | `09-key-edit.html` | 会话 |
| `/admin/users` | 15 US4 | `10-admins.html` | 会话 |

**密钥展示与复制：** 非 URL。`POST` 创建/重新生成在变更载荷中返回 `secret` → `SecretOncePanel`。列表 `GET /api/admin/api-keys` 返回 `secret`（或 `null`）供行内 **Copy**（`admin.keys.copy_list`）；表格单元格不展开完整明文。`secret == null` 时 Copy 禁用。不使用 `localStorage`。

**国际化：** 自定义目录 `messages/{EN,CN,HK,TW}.json` + `t(locale, key, vars)`。**不添加 `next-intl`。** 缺失 key → `EN` → 原始 key。从 `ui-mockup/assets/i18n.js` 初始化（去掉画廊 key）。HK 与 TW 必须有所区别。邮件复用相同文件（`admin.reset.mail_body`、`admin.users.invite_mail_body`，含 `{url}`）。邀请/重置 `{url}` 为绝对路径：依次使用 `PUBLIC_BASE_URL`、`APP_URL`，本地回退 `http://localhost:${PORT}`，生产环境为 `https://places.agent-mate.ai`。`POST /api/admin/locale` 后调用 `router.refresh()`。`html lang`：`en` / `zh-CN` / `zh-HK` / `zh-TW`。

**数据：** React Query → 同源 `/api/admin/*`，携带 `credentials: "include"`。不需要 Zustand。表单使用 RHF + Zod（包括 `/accept-invite` 向 `/api/admin/accept-invite` 的 POST）。

| 界面操作 | 接口 |
| --- | --- |
| 页头问候语 | `GET /api/admin/session` → `{ name, email, mustSetPassword }` |
| 密钥列表 | `GET /api/admin/api-keys`（含 `prefix` + 可选 `secret`） |
| 签发/重新生成 | `POST` / `POST …/regenerate` 返回 `secret` 并写入库 |
| 编辑 | 仅 `PATCH` name/description |
| 删除单个 | `DELETE /api/admin/api-keys/[id]` |
| 批量删除 | `DELETE /api/admin/api-keys`，请求体 `{ ids }`（最多 100 个） |
| 用户/邀请 | `GET /api/admin/users`、`POST /api/admin/users/invite` |
| 登录/登出/语言/密码 | 对应 `POST` 接口 |

错误格式：`{ error: { key } }`。在全部四种语言目录中新增以下 key：`admin.common.loading`、`admin.common.retry`、`admin.keys.loading`、`admin.keys.error`、`admin.users.loading`、`admin.users.error`、`admin.users.invite_sent`、`errors.session_expired`、`errors.invite_failed`、`errors.csrf`。保留原型 key（`admin.keys.empty`、`errors.login_failed`、`errors.password_required`、`admin.reset.sent`、`admin.register.disabled_prefix`、`admin.register.contact_admin`、`admin.register.disabled_suffix`、`admin.register.wechat_qr_alt`、`admin.register.wechat_qr_caption`……）。加载时不得清空页面框架。登录注册关闭提示板使用上述 key，协议为 `api-key`；资源文件 `public/EthanWeChat.png`（与 kb.agent-mate.ai 使用同一文件）。

**选择器（`data-testid`）：** `admin-home-instructions`、`admin-login`、`register-disabled`、`contact-admin`、`contact-admin-qr`、`login-submit`、`login-error`、`accept-invite-submit`、`accept-invite-done`、`accept-invite-sign-in`、`accept-invite-error`、`landing-instructions`、`admin-hello`、`nav-keys`、`nav-users`、`nav-sign-out`、`issue-key`、`keys-table`、`keys-empty`、`keys-copy-{name}`、`copy-secret`、`users-table`、`delete-admin-confirm`、`locale-EN` … `locale-TW`、`guide-capabilities`、`guide-toc-capabilities`、`guide-capabilities-table`。按行删除：`delete-admin-{id}`。

**不得**出现在客户端静态包 / `NEXT_PUBLIC_*` 中：地图密钥、`QWEN_*`、`OPENAI_*`、`GMAPS_MCP_*`、`RESEND_*`、`SESSION_SECRET`、`OPEN_METEO_API_KEY`。调用方 `secret` 仅经管理会话 API 下发至运营者后台，不写入公开 `/v1` 响应。路由处理器：`import "server-only"`。

```text
app/
  layout.tsx
  (public)/page.tsx                 # /
  (auth)/login/ reset-password/ set-password/ accept-invite/
  instructions/
  admin/layout.tsx                  # AppHeader + AppNav
  admin/page.tsx                    # redirect → /admin/api-keys
  admin/api-keys/page.tsx
  admin/api-keys/new/page.tsx
  admin/api-keys/[id]/page.tsx
  admin/users/page.tsx
  api/admin/{login,logout,session,locale,
    password/reset,password/set,
    users,users/invite,
    api-keys,api-keys/[id],
    api-keys/[id]/regenerate}/route.ts
  api/v1/...
```

左侧导航：`admin.nav.keys` → `/admin/api-keys`；`admin.nav.admins` → `/admin/users`；退出登录 `POST /api/admin/logout`。

### 12.1 视觉风格

**性冷淡**风格，与 kb.agent-mate.ai 同属一个家族：米白背景、黑色文字、细线分隔、零圆角、留白充裕。个性体现在**唯一**一处：`agent-logo.png`。

| 做 | 不做 |
| --- | --- |
| 细线分隔、等宽大写标签、黑色矩形按钮 | 阴影、渐变、圆角、彩色状态标签 |
| 用字重和下划线表示状态（激活导航、激活语言） | 左侧导航中使用图标 |
| 仅在创建/重新生成时显示一次调用方密钥 | 从列表中"再次查看密钥" |
| 四个语言代码 `EN CN HK TW` | 中文/EN 双向切换器 |

动效：公开页/认证页首次渲染时一次短暂上升（`12px`，`700ms`）。遵守 `prefers-reduced-motion`。

### 12.2 设计令牌

```css
--bg: #fafafa;  --bg-elevated: #ffffff;
--ink: #0a0a0a;  --ink-2: #1f1f1f;
--mute: #525252;  --mute-soft: #6b6b6b;
--line: #e0e0e0;  --line-strong: #bdbdbd;
--fill: #f0f0f0;  --danger: #8b1a1a;
--radius: 0;  --control-h: 2.75rem;
--font-ui: "Outfit", "Noto Sans SC", "Noto Sans TC", system-ui, sans-serif;
--font-cn: "Noto Sans SC", "Noto Sans TC", "Outfit", system-ui, sans-serif;
--font-mono: "JetBrains Mono", ui-monospace, monospace;
--max: 760px;
```

**字号规格：** 字标 Outfit 1.35/1.2rem w500；页面标题 Outfit 1.5–1.85rem w600；正文 Noto SC/TC 1.05rem；眉毛/标签 JetBrains Mono 0.75rem 大写；按钮 Outfit 0.8125rem w500；密钥/代码 JetBrains Mono 0.9–0.95rem。

**Logo：** 首页/认证页 `56×56`；页头 `36×36`；favicon `32×32` PNG + `180×180` apple-touch（透明背景）。

### 12.3 页面框架

```text
公开主页                              AUTH（登录 / 重置 / 设置密码）
┌─────────────────────────────┐     ┌─────────────────────────────┐
│                    EN CN HK TW│     │  [logo] places.agent-mate.ai │
│  [logo] places.agent-mate.ai  │     │  提示（注册已关闭）           │
│  标语 · 使用说明 · 登录        │     │  标题 · 字段 · 提交           │
│                    版权信息   │     │                    版权信息   │
└─────────────────────────────┘     └─────────────────────────────┘

指南（公开）                          APP（已登录）
┌─────────────────────────────┐     ┌─────────────────────────────┐
│ [logo] host   返回  EN CN…  │     │ [logo] host  你好，{name}    │
│ 智能体说明                   │     │ 使用说明   EN CN HK TW       │
│ 目录 · 正文 · 示意图          │     ├────────┬────────────────────┤
│                    版权信息  │     │ 密钥   │ 列表 / 表单         │
└─────────────────────────────┘     │ 管理员 │                    │
                                    │ 退出登录│                    │
                                    └────────┴────────────────────┘
```

### 12.4 组件规格

| 组件 | 规格 |
| --- | --- |
| 主按钮 | 黑色填充，白色文字，1.5px 墨色边框，圆角 0。标签为动词。 |
| 文字链接 | 墨色，1px `line-strong` 下划线；悬停 → 墨色下划线。 |
| 危险静默按钮 | 哑色文字，无填充。悬停 → 墨色。用于删除/吊销/重新生成。 |
| 输入字段 | 等宽大写标签在上；底边框输入框。填充色为 `--bg`。聚焦：2px 墨色轮廓。 |
| 通知 | `fill` 背景，1px `line` 边框。错误：`danger` 颜色文字。 |
| 表格 | 无垂直分隔线。等宽大写表头。首列为复选框列。行操作：文字链接。 |
| 密钥面板 | 等宽密钥，复制控件，一次性警告。 |
| 对话框 | 直角矩形，1px 线条，28% 墨色遮罩。Escape 关闭。 |

键盘操作：可见的 `2px` 墨色焦点环。主要操作无需鼠标即可触达。

### 12.5 页面索引

原型文件：[`ui-mockup/`](./ui-mockup/)。

| 文件 | 页面 |
| --- | --- |
| `01-home.html` | 公开主页 |
| `02-login.html` | 登录；`?error=1` 失败 |
| `03-reset.html` | 重置申请；`?sent=1` |
| `04-set-password.html` | 重置 / 空密码设置；`?done=1` |
| `14-accept-invite.html` | 接受邀请引导；`?done=1` |
| `05-instructions.html` / `11-instructions-app.html` | 指南（公开 / 已登录） |
| `06-keys.html` | 密钥列表；`?empty=1` |
| `07-key-new.html` / `08-key-created.html` | 创建密钥 / 一次性密钥 |
| `09-key-edit.html` | 编辑 / 确认重新生成或删除 |
| `10-admins.html` | 管理员 + 邀请 |
| `12-email-reset.html` / `13-email-invite.html` | Resend 邮件模板 |

### 12.6 文案规范与质量标准

- 按运营者的操作命名控件（签发、复制、重新生成、删除、邀请）。
- 错误信息指明失败原因和下一步操作。不做无意义的道歉。
- 界面中不提及地图供应商密钥、Portainer 或 OPENAI_CN。
- 桌面端（约 1280px）和移动端（约 390px）：公开页/认证页列可读；应用导航 → 文字菜单。
- 焦点顺序：跳过链接 → 语言选择 → 主要区域 → 主操作。
- 生产环境中密钥绝不写入 `localStorage`。

---

## 13. 本地化流水线（工具 + 界面）

| 层 | 机制 |
| --- | --- |
| A 文案 | 目录 `EN` `CN` `HK` `TW` |
| B 地点名称 | 供应商 `languageCode`；不用术语表重写 |
| C 数字 | `Intl`；货币来自地点所在国家 |
| 天气 | `weather_code` → `weather.wmo.{code}` — **非** B 层 |
| L LLM 文本 | locale 指令 + 术语表（`HK`/`TW` 时） |

不得通过 `t()` 处理的内容：`places-agent`、locale id、供应商 id、主机名、`admin` / `me@ethanhuang.com`、工具名称、`Authorization: Bearer`。

---

## 14. 环境变量

`3.tech-specs.md` 中列出的变量均需配置。本进程还额外需要：

```env
DATABASE_URL=postgresql://…@101.132.156.250:5432/places_agent
PORT=3000
PROMPT_ID=chat.v1
GLOSSARY_ID=
CATALOG_PACK=catalogs.v1
# MVP-11 — Orizn Visa（operator-owned；见 ADR-044）
ORIZN_API_KEY=orizn_visa_…
ORIZN_VISA_BASE_URL=https://visa.orizn.app/api/v1
ORIZN_CACHE_TTL_H=24
```

**MVP 不使用：** `OPENAI_IMAGE_MODEL` 作为图像生成工具。对话仍可接受图像**输入**。**绝不：** `NEXT_PUBLIC_*` 暴露密钥。

---

## 15. 测试与 Makefile

遵循 [`agent-test-plan.md`](./agent-test-plan.md) 及通用测试策略。仓库内：Vitest + RTL + Playwright（`3.tech-specs.md`）。工具双渠道契约测试；管理后台 E2E 测试覆盖功能 14–19。CI 默认仅使用 Fixture。

脚手架：根目录 `Makefile`，包含 `dev` / `up` / `down` / `test` 目标。`dev` 运行 `server.ts`。

---

## 16. 渐进构建顺序

按**智能体能力**划分为两个产品切片 — 见 [`agent-stories.md`](./agent-stories.md) MVP 计划。在 MVP-2 之前完成 **MVP-1**（包括**全部管理后台 UI 14–19**）。

**MVP-1 — 运营、调用、搜索餐厅**（每次一个用户故事）：14 主页 → 15 登录/用户 → 16 落地页 → 19 国际化 → 18 使用说明 → 17 调用方密钥 → 12 调用方密钥认证 → 11 HTTP+MCP（`server.ts`、`/v1/health`、`/mcp`）→ 6 `providers[]` → 5 地理编码 → 1 `search_restaurants`（先 HTTP 后 MCP）→ 3 详情 → 7 `sources[]` → 4 导航 → 13 工具语言（天气 key 等待功能 9）。

**MVP-2 — 地点、行程、富化、对话：** 2 `search_places` → 9 `plan_itinerary` + Open-Meteo（`weather.wmo.*`）→ 8 Tripadvisor 富化 → 10 自然语言对话循环（复用工具核心）。

---

## 17. 反模式

- 重新创建 `agent-config/geo-capability-route.json`
- 使用 `NEXT_PUBLIC_` 暴露地图密钥
- 以管理员会话作为工具凭证
- 在 `CN`/`HK`/`TW` 中使用英文 Open-Meteo 短语
- 仅 Mock 的地图适配器标记为已完成
- 将 AMAP-agent / Google-agent / weather-agent 用作 LLM 智能体
- MCP 边车
- 在英文 locale 查询字符串中硬编码中文关键词
- 使用硬编码的城市景点列表而非通用模板
- 返回空的 `photos: []` 而非省略该字段
- 混合语言的搜索查询（例如在一个查询中同时使用 `"cafe tea house"` 和 `"咖啡馆"`）
- CJK 启发式误判海外中文城市名（新加坡、大阪、曼谷）为大陆 — 应使用排除列表
- Fixture 地理编码将所有未知城市解析为香港默认值 — 应扩大覆盖范围

---

> **归档说明（2026-09-11）：** 原 §18（MVP-10 轻骨架 + 增量无 LLM 填充工具族）、§20（MVP-12 必去地统一获取 + travel_tips + MCP 无会话化）的 as-built 细节已移至 [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md)。这两段被 ADR-067 LLM 驱动发现与 MVP-T3/T3++ 路径取代；本文件仅保留当前 live 架构（§19 Visa、§21 Trip Store）与 T3/T3++ 设计。

---

## 19. Orizn 签证 adapter + `visa_requirement` 工具（MVP-11，2026-09-01 已实现）

真源：[ADR-044](../adr/ADR-044-orizn-visa-rest-adapter.md) · Feature **48** · where2play Feature **38–39**。

### 19.1 架构定位

- **辅助数据源**（与 Open-Meteo 同类）：不进 `providers[]`、不参与 ADR-026 区域路由。
- **传输：** REST 直连 Orizn `GET /api/v1/visa`；密钥 `ORIZN_API_KEY` 仅存在于 places-agent 进程。
- **不采用：** stdio 子进程 `npx orizn-visa-mcp`；远程 `https://visa.orizn.app/mcp`（实测仅 `quick_visa_check`，无完整材料/流程）。

### 19.2 模块布局（目标）

```text
src/adapters/orizn/
  config.ts      # ORIZN_API_KEY, ORIZN_VISA_BASE_URL, ORIZN_CACHE_TTL_H
  direct.ts      # fetch GET /visa?passport=&destination=&lang=; injectable FetchFn
  fixture.ts     # CHN→JPN, CHN→SGP 等固定样本
  live.ts        # PLACES_VENDOR_MODE=live 组装
src/core/visa-requirement.ts   # visaRequirement(input) → VisaRequirementResult
```

HTTP：`POST /v1/visa_requirement`（[`dispatch.ts`](../src/http/dispatch.ts) + [`schemas.ts`](../src/http/schemas.ts)）。  
MCP：[`create-server.ts`](../src/mcp/create-server.ts) 注册 `visa_requirement`（Zod inputSchema，与 HTTP 共享 core）。

### 19.3 输入 / 输出契约

**输入：**

```ts
{
  passport: string,       // ISO 3166-1 alpha-3，如 CHN
  destination: string,    // ISO 3166-1 alpha-3，如 JPN
  locale?: "EN"|"CN"|"HK"|"TW"  // 缺省 EN
}
```

**Orizn lang 映射：** EN→`en`；CN/HK/TW→`zh`（Orizn 无 zh-HK/zh-TW 变体）。

**输出 `data`（节选）：**

```ts
{
  passport, destination,
  requirement: "visa_free"|"visa_required"|"e_visa"|"visa_on_arrival"|"eta"|"no_admission"|...,
  visa_free_days: number | null,
  description?: string,
  documents?: string[],
  process?: string[],
  processing_time?: string,
  validity?: string,
  max_stay?: string,
  extension?: { possible: boolean; details?: string },
  last_verified?: string | null,
  source_url?: string | null,
  unavailable_fields?: string[]   // 免费档 upgrade 占位字段名
}
```

### 19.4 错误与配额

| 条件 | 行为 |
| --- | --- |
| 非法 alpha-3 | `errors.visa_invalid_country_code` |
| 缺 `ORIZN_API_KEY`（live） | `errors.visa_unconfigured` |
| Orizn 403/429 | `errors.visa_quota_exceeded`；不编造签证事实 |
| 缓存命中 | 同 `(passport, destination, lang)` TTL 内不重复请求 |

### 19.5 where2play 消费（后续切片）

- **本 MVP-11 agent 切片：** 仅交付工具；where2play **不**在本切片开发查询 UI。
- **where2play MVP-11（spec）：** 注册/资料页增加 `nationality`（ISO alpha-3，选填）；Prisma `User.nationality String?`。
- **出行建议页 / Plan 贴士签证卡（规划中）：** BFF 读 `User.nationality` + 目的地 → `POST /v1/visa_requirement` **写入** `artifacts.visa`；**UI 经 `fetch_trip_details` `artifacts` 展示**。禁止把 visa HTTP 响应当 2play 渲染源（Feature **39** 占位，实现待后续立项）。

---

> **归档说明（2026-09-11）：** 原 §22（MVP-18 规划主干读模型 + artifacts）、§23（宿主生成行程调用契约）、§24（MVP-19 超时 / 热度打标 / 骨架硬闸）、§25（MVP-23 规划行程细节 fill）、MVP-T1 as-built、MVP-T2 as-built 已移至 [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md)。这些被 MVP-T3 skeleton-first 路径与 MVP-T3++ 智能体规划行程核心机制取代；artifacts 四卡推迟到 T7，fill/meals/directions 推迟到 T5+。

---
