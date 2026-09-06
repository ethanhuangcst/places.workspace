# ADR-045: 必去地统一获取（双模）+ travel_tips + 工具清理 + MCP 无会话化

## Status
Accepted（2026-09-01）。部分 supersede [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) Update 节的「未决技术债：LLM 推断必去是过渡方案」——本 ADR 把必去地获取从 discover 内联提升为独立双模方法 `findIconicPlaces`，并新增 `travel_tips` 工具复用它。**不**废止 ADR-042 的源码禁城市知识原则；ungrounded 模式的知识仍在 LLM 权重，不在源码。

本 ADR 同时合并 **MCP 无会话化（stateless）** 决策，解决 `mcp_session_invalid` 导致宿主 LLM 编造行程的可用性问题。

**Update 2026-09-03（MVP-19 P0c）：** Decision §2 / §3 中「discover 并行 ungrounded LLM + 单布尔 `must_see` 兼用户必去」由下文 **Update（MVP-19 池后热度 + 正交字段）** 取代。`findIconicPlaces` 仍不升 HTTP。`travel_tips` / MCP 无池路径不变。

## Context

MVP-10 落地后，必去地（must-see）获取存在三处碎片化：

1. **推断**：`inferMustSeeFromPool`（`discover-must-see-llm.ts`）在 `discover_places` 末尾从候选池挑 ≤3，池校验。上限 `MAX_MUST_SEE=3` 硬编码。
2. **合并**：`dedupeMustInclude`（`create-server.ts:38`）把用户 `must_include` 与推断结果合并——**只在 MCP discover handler**，HTTP/BFF 路径不合并。
3. **下游对账**：`make_itinerary` / `arrange_day` 收到 `must_include: string[]`，需与池按名匹配才能排程；名字形态对不上（"Jerónimos Monastery" vs "Mosteiro dos Jerónimos"）会触发 `validateSkeleton` 硬失败重试。

新需求 `travel_tips`（目的地 tips：80 字介绍 + 必去 top3 + 交通 + 天气 + 着装 + 安全）可**独立于其他能力**调用，**可能在 `discover_places` 之前**——即没有候选池时就要给出必去地。现有 `inferMustSeeFromPool` 强依赖池（无池返回 `[]`），无法满足。

ADR-042 的"未决技术债"明确：LLM 推断必去是过渡方案，热度包落地后可下线。本 ADR 不改这一方向，只把过渡方案统一化并支持无池场景。

### MCP 会话可用性问题（合并入本 ADR）

实测 ChatBox 经 MCP 调 `discover_places` 时出现 `mcp_session_invalid`，宿主 LLM 在工具失败后**自行编造**自由文本行程。根因有三层：

1. **服务端重启清空内存会话**：开发期频繁重启 + 生产每次部署都清空 `mcpSessions`，所有客户端会话失效。
2. **30 分钟空闲 TTL 过期**：客户端闲置超 TTL 后再发请求，会话已过期。
3. **客户端不自动重初始化**：服务端已按协议返回可恢复错误，但 ChatBox 不执行 re-init，直接把失败交回宿主 LLM。

关键发现：places-agent 的 **MCP 传输会话不承载任何工具所需状态**——`must-include-coverage` 与 `arrange-present-gate` 均按 `city|origin|locale` 独立键存于各自的内存 Map，与 MCP 会话 id 无关；MCP 工具结果在 `tools/call` 响应内同步返回，不依赖服务端发起的 SSE 通知。即会话对工具是纯协议开销。

## Decision

### 1. 新增 core 方法 `findIconicPlaces`（双模，不单独注册 MCP/HTTP）

```ts
// core/find-iconic-places.ts
export type FindIconicPlacesInput = {
  destination: string;
  pool?: PlaceCard[];        // 有则 grounded，无则 ungrounded
  limit: number;             // 硬上限，调用方传
  locale?: Locale;
  _testChatCreate?: ItineraryChatCreate;
};

export type FindIconicPlacesResult = {
  names: string[];
  grounded: boolean;         // true=池校验可排程；false=纯 LLM 仅展示
};

export async function findIconicPlaces(
  input: FindIconicPlacesInput,
): Promise<FindIconicPlacesResult>;
```

- **pool 非空 → grounded（现行）**：对已有池按热度打标（`user_ratings_total`，缺则 `rating`），**不**再搜供应商、**不**走 LLM。返回 `grounded: true`，并写 `must_see`。名字可进 `make_itinerary`。
- **pool 空 → ungrounded**：LLM 按目的地参数化生成（prompt 含目的地名，因不排程无对账问题）。返回 `grounded: false`，仅供展示。
- 现有 `inferMustSeeFromPool` 退化为 grounded 分支内部实现；`dedupeMustInclude` 从 `create-server.ts` 下沉到 core，两模共用归一化去重 + limit 截断。

**ADR-042 合规：** ungrounded 模式知识在 LLM 权重，源码无城市表；grounded 模式沿用现有池校验。两模均不把"城市 → POI 列表"写进源码。

### 2. `discover_places` 改造：并行 + 补搜 + must_see 标志

```
Phase A（并行）:
  findIconicPlaces(destination, limit=3)  → iconic 名（ungrounded LLM）
  searchCandidatePools(类目 job)          → 类目池（vendor，0 LLM）
Phase B（A 完成后）:
  匹配 iconic ∪ user_must_include against 类目池 → matched / unmatched
Phase C（补搜保证进池）:
  对 unmatched 名做 name-search job（并行，少量）→ 补搜结果
  合并去重进池
Phase D:
  命中卡片打 must_see=true
  返回 { candidates: pool(带 must_see), inferred_must_see }
```

- **并行**：Phase A 两路并行，墙钟 ≈ max(LLM ~10s, vendor ~3-5s) ≈ 10s；Phase C 补搜 ~2-3s。总 ≈ 12-13s，优于现状 ~13-15s。
- **补搜保证进池**：unmatched 必去名一律补搜一次，仍搜不到才丢弃（记日志）。消除"discover 漏搜导致必去地排不进"根因。
- **iconic 补搜不替搜**：类目搜索保留（保证多样化候选池），iconic 名搜索为补充。

### 3. `PlaceCard` 加 `must_see` 字段

```ts
// core/types.ts
must_see?: boolean;   // true=必去（LLM iconic ∪ user must_include 合并）
```

- 非破坏性，只在 discover 流程内赋值，其他来源默认 undefined。
- `make_itinerary` / `arrange_day` 直接读 `candidate.must_see`，不再需独立 `must_include: string[]` 对账回合。
- 展示层（where2play stop 卡、travel_tips）可直接渲染"必去"徽章。
- 一期单标志不分来源；日后需要再加 `must_see_source: "llm" | "user" | "both"`。

### 4. 新增 `travel_tips` 工具（HTTP + MCP）

```ts
// 输入
{
  destination: string;
  bounds?: { start: string; end: string };   // 有则真实天气，无则气候平均
  trip_type?: string;                         // 出行类型
  pace?: string;                              // 出行节奏
  skeleton?: ItinerarySkeleton;               // 有则提取 stops 名作 pool（grounded）
  constraints?: Record<string, unknown>;      // where2play 行程规划输入的其他限制
  locale?: Locale;
  providers?: string[];
  pool?: PlaceCard[];                          // 显式池；与 skeleton 二选一
}
```

**输入处理（方案 A）：** 若传 `skeleton`，travel_tips 内部提取 `skeleton.stops[].name`（attraction 类）组装 `pool: string[]` 传给 findIconicPlaces（grounded）。提取为纯数据转换，无 LLM。where2play 只传 skeleton 对象，不碰 LLM。`skeleton` 与显式 `pool` 二选一；两者都传时 skeleton 优先。

流程（2 次 LLM，关键路径 ≤ 20s）：

```
┌─ geocode(destination) ──→ weather × N (parallel) ──→ aggregate ─┐
│                                                                  ├─→ merge ──→ tips-prose LLM ──→ result
└─ findIconicPlaces({ destination, pool, limit:3 }) ──────────────┘
```

1. **geocode**（0 LLM）→ 坐标
2. **weather**（0 LLM，open-meteo）→ 行程期间逐日预报，并行取
3. **findIconicPlaces**（1 LLM）→ 必去 top3；有 skeleton→grounded，无→ungrounded。**不做二次供应商验证**，信任返回结果；ungrounded 返回 `grounded:false` 供展示层标注「未验证」。
4. **tips-prose LLM**（1 LLM）→ destination + 聚合天气 + iconic 名 + trip_type/pace/constraints → 80 字介绍 / 交通 / 着装 / 安全
5. 返回结构化 + 短文本

**LLM 调用 = 2 次**（findIconicPlaces + tips-prose）。travel_tips 低频信息工具，2 次可接受。MCP 场景宿主可再渲染，但工具返回成稿，HTTP 也能直接用。

#### 4.1 天气聚合规则

多日行程，逐日预报聚合成单段天气上下文喂 tips-prose：

- **severity**：取全段最差值（`fair < caution < adverse < severe`），代表「行程期间最不利天气」。
- **drivers**：全段并集去重（如 `rain ∪ wind`），代表「行程期间需应对的全部天气因素」。
- **temperature**：min/max 区间（`[min(逐日最低), max(逐日最高)]`）。
- 单日行程直接用当日预报，不聚合。

#### 4.2 20s 超时与降级（硬保证）

外层 `AbortSignal.timeout(20_000)` 包整个 travel_tips；内层分步超时与降级：

| 步骤 | 超时 | 超时/失败降级 |
|------|------|--------------|
| geocode | 3s | 失败 → 跳过 weather，tips-prose 无天气数据（提示「天气暂不可用」） |
| weather | 3s（并行总） | 失败 → 同上，降级不报错 |
| findIconicPlaces | 12s | 超时 → 降级：tips-prose 不带 iconic 名，自生成（`grounded:false`） |
| tips-prose | 10s | 超时 → 报错 envelope（`errors.travel_tips_timeout`） |

**降级原则：** weather 和 iconic 可降级（缺了仍能给 intro/交通/着装/安全）；tips-prose 不可降级（核心输出）。

**LLM 参数：** findIconicPlaces `max_tokens 300, temperature 0.3`；tips-prose `max_tokens 900, temperature 0.4, stream:false`。两者均 `AbortSignal` 硬中断。

**缓存：** geocode 复用 `cachedGeocode`；weather 按 `{lat,lng,date}` 缓存 30 分钟；findIconicPlaces 按 `{destination, pool-hash, limit}` 缓存 1 小时。

**墙钟估算：** 关键路径 = max(geocode+weather, findIconicPlaces) + tips-prose = max(0.7s, 8s) + 8s ≈ 16s，留 4s 余量。

### 5. 别名重指向（保留命中 MCP）

`plan_itinerary` / `trip_plan` / `trips` 三个别名**保留注册**，handler 从旧 one-shot `planItinerary` **重指向 `make_itinerary`**（参数适配）。用户说"plan a trip"/"行程"/"trips"仍命中，落到新骨架管线。

### 6. 删除（gate：where2play plan-46）

| 工具 | 处置 | gate |
| --- | --- | --- |
| `arrange_day` | 删 | where2play BFF `plan-day-by-day.ts` 切新管线后 |
| `enrich_arrange_transit` | 删（HTTP-only） | 同上 |
| 旧 `planItinerary`（core/itinerary.ts） | 删 | 别名重指向后 |

`navigate` 已删（F45 部分 Done）。本 ADR 的 agent 侧改造（findIconicPlaces、travel_tips、discover 改造、must_see 标志、别名重指向）**不依赖** plan-46，可先行；硬删除等 plan-46 切完。

### 7. MCP 无会话化（stateless）

`/mcp`（Streamable HTTP）传输改为 **stateless 模式**：`sessionIdGenerator: undefined`。

- 响应**不**返回 `mcp-session-id` 头；**不**做会话校验；每个请求独立。
- 消除 `mcp_session_invalid` 错误类（不可达）——无会话则无过期、无重启清空、无需客户端 re-init。
- 实现采用**单例共享 stateless transport**（启动期 lazy 创建一次 `StreamableHTTPServerTransport({ sessionIdGenerator: undefined })` + `createPlacesMcpServer()` + `server.connect(transport)`，所有 `/mcp` POST 复用），避免每请求重建 server 的开销。SDK 明确支持 stateless 复用（无每会话状态）。
- GET（SSE 上行流）/ DELETE：places-agent 的 MCP 不用服务端发起通知，stateless 下 GET 返回 `405 Method Not Allowed`；DELETE 无会话可删，返回 `200` 空体或 `405`。
- 旧 `/sse`（legacy SSE 传输）保持现状不动（向后兼容），不在本 ADR 范围。

**安全性**：无会话不削弱鉴权——places-agent 的 MCP 端点鉴权（若有）由部署层（网关/token）承担，与会话无关；工具本身无鉴权差异。

### 8. 宿主指令强化（防编造，兜底层）

即便 stateless 消除了主因，其它失败（provider 超时、限流）仍可能让宿主 LLM 编造。在 `host_instructions`（F47）追加**硬约束**：

> 任何工具调用失败（会话失效、provider 错误、超时）时，**禁止**用参数知识编造行程/地点/交通。必须告知用户规划服务暂不可用并请其重试；可改用 `travel_tips` 给出目的地通用信息作为降级，但不得伪造具体行程。

这是兜底层，与 stateless 互补：stateless 治本（消除会话失败），指令治标（其它失败时防最差行为）。

## Rationale

- **双模必要性**：travel_tips 可在无池时调用，纯池锚定方法无法满足；而排程场景仍需池锚定保证可调度性。一个方法两模避免逻辑重复。
- **并行+补搜**：并行省墙钟，补搜保证必去地进池——两者解决不同问题（速度 vs 覆盖），不互斥。
- **must_see 标志**：信号跟卡片走，消除下游名字对账回合，展示层直接可用。
- **别名保留**：MCP 命中率，避免用户自然语言失配。
- **不改 ADR-042 方向**：热度包仍是真正替代；本 ADR 只统一过渡方案并扩到无池场景。
- **stateless 可行性**：工具状态键独立于 MCP 会话，会话对工具是纯开销；SDK 原生支持 stateless。arrange_day 删除后连"顺序展示"这一会话粘性理由也消失，stateless 是自然终态。
- **治本优于治标**：stateless 消除会话失败根因，优于"持久化会话"（传输对象不可序列化，重启后连接已断，不可行）或"靠客户端重试"（客户端行为不可控）。host_instructions 仅作兜底。

## Consequences

- `discover-must-see-llm.ts` 的 `inferMustSeeFromPool` 被新 `find-iconic-places.ts` 取代（grounded 分支复用其逻辑）。
- `create-server.ts` 的 `dedupeMustInclude` 下沉到 core。
- `PlaceCard` 类型加可选字段，非破坏性。
- `make_itinerary` / `arrange_day` 改读 `candidate.must_see`（arrange_day 在删除前过渡期仍兼容 `must_include` 入参）。
- 新增 `travel_tips` HTTP route + MCP 注册 + i18n keys + 测试。
- 删除动作 gate 于 where2play plan-46；agent 侧可先行。
- **MCP `/mcp` 改 stateless**：`http-transport.ts` 的 `handleMcp` 改用单例 stateless transport；移除 `/mcp` 路径的 `mcpSessions`/`SessionManager` 依赖与 `mcp_session_invalid` 分支（保留 `/sse` legacy 不动）。`session-manager.ts` 若仅 `/sse` 使用则保留，否则评估下线。
- `host_instructions`（F47）追加"工具失败禁编造"硬约束。
- 更新 `2.architecture.md` ADR 表、`agent-stories.md`（Feature 49/50/51/52）、`agent-design.md` §20、`agent-test-plan.md` §21。

## Update（2026-09-02 · 2play 读模型）

[ADR-046](./ADR-046-trip-store-pg-memory-fetch.md) 落地后：**2play 不把本 ADR 的 `travel_tips` HTTP 成稿当 UI 真源。** Agent 仍可在写路径调用 `findIconicPlaces`（及可选 tips-prose）并把结果写入 Trip `artifacts.tips`。where2play 展示只 `fetch_trip_details`。tips-prose 超时不得抹掉已得 `iconic_places`（HTTP 200 + 双写）。签证展示同理：Orizn → `visa_requirement` 写 `artifacts.visa` → fetch。详见 knowledge `itinerary-ui-fetch-only.md`。

## Update（2026-09-03 · MVP-19 池后热度 + 正交字段）

**范围：** 取代本 ADR Decision §2（discover 并行 ungrounded + 名补搜打标）与 §3（单布尔兼用户必去）。不新开 ADR 号；不升 HTTP `find_iconic_places`。实现：Feature **79**（P0b）、Feature **82**（P0a）。规格：`agent-design.md` §20.4 / §20.5。

### 为何改

里斯本 3 日复现：`findIconicPlaces` 与类目搜并行时，芯片名单来自无池 LLM，与供应商池热度脱节；`must_include` 再把用户 3 处写回同一 `must_see`，AND `make_itinerary` 用瘦身 body 覆盖 `Trip.candidates`，热门标被折叠。ADR-042 禁止用城市百科补名单。

### 决策

**D1 — discover 相位（2play 起飞路径）**

```
Phase A: searchCandidatePools（类目供应商搜；0 LLM）
Phase B: 对 attraction 池按 user_ratings_total 降序（缺则 rating）打前 iconicLimitForTripDays 张 must_see=true
Phase C: 若请求带 must_include：补搜进池 + user_requested；不得把已有 must_see 设为 false
Phase D: 双写 candidates（slim 保留 rating / user_ratings_total）；信封 inferred_must_see = 池内 must_see 名（热度序）
```

2play 起飞 discover **不带** `must_include`。墙钟 ≈ 供应商搜。禁止为打标再跑一轮无池 LLM。

**D2 — `findIconicPlaces` 角色收窄**

- **仍是** core 方法：有池 = 热度打标（discover Phase B）；无池 = `travel_tips` ungrounded LLM。
- 2play 步骤 g 芯片 = fetch `candidates` 上 `must_see === true`（热度序），**不是** ungrounded 名单。
- **不**注册 HTTP/MCP 工具 `find_iconic_places`。
- **不**为打标再调供应商 nearby / 热点搜。

**D3 — 正交字段（取代 §3 单布尔合并）**

| 信号 | 存放 | 含义 |
| --- | --- | --- |
| 热门 / iconic | `PlaceCard.must_see` | 仅 Phase B 热度（及既有热门保留合并） |
| 用户指定 | `Trip.constraints.must_include: string[]` | 用户原文名单 |
| 用户命中卡 | `PlaceCard.user_requested?` | 可选；只加不减 `must_see` |

写 `candidates` 时按归一化名 **merge-preserve**：库中 `must_see === true` 不得被瘦身补丁清掉。不新建 POI 表、不扩 CATALOG。

**D4 — 明确不做（本补丁）**

HTTP `find_iconic_places`；城市百科；独立 POI 表；用用户 3 处覆盖池上 8 处 `must_see`。

### 后果

- Decision §2 并行 LLM + iconic 名补搜：**discover 主路径下线**。
- Decision §3「一期单标志不分来源」：**废止**；来源拆到 `must_include` / `user_requested`。
- `travel_tips` 仍可 ungrounded `findIconicPlaces`（无池展示）；2play Plan 芯片不以该名单为真源（续 2026-09-02 Update）。
- 热度包 / 外部 publishable pack 仍是 ADR-042 允许的远期替代；本补丁用供应商评论数作过渡，不把城市→POI 写入源码。

## Update（2026-09-03 · F41 S2）

2play Plan **init** 路径：discover **先类目搜建池，再** 内部 `findIconicPlaces({ pool, limit: max_number 默认 5 })` **按热度从该池打** `must_see`。禁止再调 API/LLM 搜附近热点或池外提名。仍不升 HTTP。芯片 = fetch 后的 `must_see` 名。`travel_tips` 无池路径不变。

## Date
2026-09-01；Update 2026-09-02；Update 2026-09-03（MVP-19 P0c / F41 S2）
