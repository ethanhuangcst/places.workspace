# ADR-043: ChatBox MCP 与跨产品关闭包（双通道时刻/交通 + L1 热门）

## Status
Accepted（2026-08-23）— 供 **places-agent** 与 **where2play** 同步的合并决策真源

## Context

Lisbon ChatBox 实录暴露：宿主显式 `execution=host`、自拟多问问卷、批量 `arrange_day`、散文无 `start_time`/无腿；同时 ADR-038 城表无法覆盖 Lisbon（[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)）。多份 ADR（037/038/040/042）与 `performance.md` / 2play specs **as-built 漂移**（[ADR-039](./ADR-039-cross-product-as-built-vs-target.md)）。

本 ADR **合并**未决与未关项，固定双通道契约，避免再散落「Directions 待决」。

## Decision

### D1 — 双通道（时刻与交通）

| 通道 | 排程谁跑 | 时刻从哪来 | 交通如何返回 |
| --- | --- | --- | --- |
| **ChatBox MCP** | places-agent `arrange_day` **强制 agent** | `blocks[].start_time` + `duration_min` | **同一响应**内 Feature 37：`blocks[].legs_to_here`（GMAP/AMAP Directions；失败 heuristic + outcome） |
| **2play HTTP** `/api/plan` | Mode H：`execution=host` → **2play OPENAI_CN** | LLM 日 JSON 的 `start_time` / `duration_min` | **另调** `POST /v1/enrich_arrange_transit` → `legs_to_here` → UI transit slots（**不是**再交交通 prompt） |

### D2 — MCP 强制 agent（升级 ADR-040 D4'）

1. MCP `arrange_day`：**忽略**调用方 `execution=host`，一律 `agent`。  
2. HTTP `/v1/arrange_day` 仍允许 `execution=host`（2play Mode H 不变）。  
3. **禁止**要求终端用户改 ChatBox system prompt。

### D3 — 聊天侧收齐边界 + 按天上屏（升级 ADR-040 D3；Option A）

1. **默认：** 宿主在对话里收齐 city / 开始日 / 天数（及每日酒店），**再**一次性调用 `discover_places` / `arrange_day`。禁止「调 MCP → 问一题 → 再调 MCP」问卷循环。  
2. **安全网：** 若仍缺字段，返回 `need_input` + `questions[]` / `remaining_fields` + `host_action: ask_in_chat_then_call_once`（一次列出缺口；聊天问完再 **call once**）。  
3. 成功日结果带 `next_action`：非末日为 `present_day_then_continue`（富日卡上屏后 **IMMEDIATELY** 调下一 `dayIndex`，**不要**等用户「继续」）；末日为 `present_day_then_overview`。  
   - **禁止**「同回合静默排完 N 天再一次倾倒」——须先写出可见日卡，再立刻排下一天（可同回合多工具，但每轮先呈现）。  
   - **`num_days`：** 每次 `arrange_day` 应传总天数；`host_instructions` 明确写 `Day N of M` + IMMEDIATELY 下一天，或 `LAST day` + 总览后停止。禁止从 Day 1 重跑 / 重 discover。  
   - **瘦身：** MCP `discover_places` / `arrange_day` 响应与入参候选须 slim（`photos[0]`、去 hours、arrange 结果去掉 `photos`/`photos_cover`）。`candidates.sources` **必须是数组**；宿主若改写成对象，服务端 normalize，不得因 `.map` 崩栈。  
   - **每日酒店可选：** 聊天应询问，但不因缺失而 `need_input` 卡死。无 origin → 省略 `from_origin`/`to_destination`，行程自首 block 至末 block；站间仍须 `legs_to_here`。  
   - **固定 8 行行程表：** 城市、开始日、天数、可选酒店、节奏（轻松/适中/紧凑）、消费 `spend_level` 1 节约 / 2 适中 / 3 宽松（默认 2）、兴趣、必去地名。禁止随机少问。  
   - **排满：** 默认 `pace=medium` 须含晚餐并收工约 ~20:00；末块结束早于 16:00 视为不满。  
   - **必去覆盖闸：** 见 **D7**（硬覆盖在共用 `arrangeDay`；`day_theme` 文案不算 covered）。日卡须多行块（禁单行 `|`）。  
4. **软闸：** 同对话键上一日未确认展示又调更大 `dayIndex` → `need_present_previous_day`。确认：`presented_previous_day` / `ack_day_index`（自动续排时宿主应在下一调用带上）。

### D7 — must_include 硬覆盖（共用核心；HTTP = MCP）

Lisbon ChatBox 复现：宿主写 `day_theme=辛特拉` 却排克卢什/Fronteira，或 `卡斯凯什` 主题却只有贝伦；旧闸用主题子串记 covered → 假覆盖。

1. **唯一实现面：** `arrangeDay`（及抽出的纯函数模块）。HTTP `/v1/arrange_day` 与 MCP `arrange_day` **同路径、同字段** `must_include_coverage`。禁止 MCP-only 搜点/覆盖逻辑。  
2. **`day_theme` 永不单独记 covered。** 空 blocks 永不 covered。  
3. **自动补搜（一次一个）：** 当 `preferences.must_include` 仍有 missing 时：若 `day_theme` 对齐某个 missing token 则选该 token，否则按名单顺序选第一个 → `geocode` 锚点 → `searchPlaces` → **合并**进候选（不替换城内点）→ prompt 标记 HARD MUST SCHEDULE（LLM 可混排半天/几点）。  
4. **硬覆盖：** 优先：block 的 `name` / `native_id` 属于该 token 当次 **search 必去 pool**。辅证：落在锚点 **10km** 内 **且** name/address 含 token 或 geocode 别名。仅距离、仅 `day_theme`、空字符串匹配 **不算** covered（避免克卢什≈辛特拉、贝伦≈卡斯凯什假覆盖）。  
5. **末日：** 仍有 missing → MCP `present_day_then_cover_must_include`（禁总览）；HTTP 返回同一 `must_include_coverage`。  
6. **守 ADR-042：** 无城百科；锚点与 POI 来自 geocode/search。  
7. **短句命中工具：** 加强 `discover_places` / `arrange_day` 等工具 description（城市+天数/行程须先调本工具）；不依赖改 ChatBox system prompt。

### D8 — 空候选服务端自动 discover（对齐硬必填）

Lisbon ChatBox 实录：宿主已有 `discover_places` 池却用 `places: []` / `restaurants: []` 调 `arrange_day` → 校验失败；错误文案「Fix candidates」又推宿主去 `search_places` 手编 POI。

1. **硬必填仅：** 城市 + 开始日期 + 天数（intake）。酒店 / 节奏 / 消费 / 兴趣 / 必去 / **调用方候选池** 均可选。  
2. **共用 `arrangeDay`：** 经 `exclude_names` 过滤后若 **`places` 为空**，且 `city` 已给 → **服务端调用 `discoverPlaces`（或测试注入）** 填景点池（餐厅侧若亦空则一并填），再进 must_include（D7）与 LLM。仅 `restaurants` 为空而 `places` 已有时不触发 live discover。禁止依赖宿主 invent POI。  
3. **无 city 且池空：** 清晰失败（须 city 或非空候选），不得诱导手编景点。  
4. **错误 / 工具文案：** 失败时说明「空池由服务端按 city 自动补；勿 invent；同 dayIndex 重试」；禁止「Fix candidates → host search」主路径。  
5. **展示：** 末日 `present_day_then_overview` — Day 卡与总览各写 **一次**后 STOP；禁止重贴日卡/总览；日卡仅列 `blocks[]`（禁自造自由漫步块）。Intake：`intake.question` **原文粘贴**，勿改写第 7 行兴趣示例。

### D9 — must_include 强制排日（P0）+ 末日卡单次展示（P1）

D7/D8 后仍复现「必去写了却没排」：根因是 `must_include` 只有「搜入候选 + prompt」，[`ensureHardMustSeeCoverage`](../../1.places-agent/src/core/must-see-coverage.ts) 只注入西安 cluster，不注入 traveler `focusPool`；且允许 `ok:true` + `missing` 非空。末日 Day 卡双写则是宿主同轮把日卡+总览贴两遍，服务端零信号。

**P0 — must_include 强制排日（关 open issue #2）：**

1. **会话日计划：** [`must-include-coverage.ts`](../../1.places-agent/src/core/must-include-coverage.ts) 会话增 `assignment: token → dayIndex`。`numDays >= N` 时一对一天；不够则从前几日塞满。`must_include[]` 首次非空写入后 **sticky**（host 后续省略仍生效）。
2. **seed 时机：** `discover_places` 收到 `must_include` 时 seed 会话（今日 MCP 路径丢名单 — 必须修）；首次 `arrange_day` 带非空名单时亦 seed。
3. **每日 focus：** 优先本 `dayIndex` 的 assigned missing token，仍只搜 **一个** focus/日。geocode + `searchPlaces` + `mergeMustIncludeIntoCandidates` 保留。
4. **确定性注入 `ensureHardMustIncludeCoverage`：** 镜像 `ensureHardMustSeeCoverage`，挂在 [`itinerary-planner.ts`](../../1.places-agent/src/core/itinerary-planner.ts) `ensureHardMustSeeCoverage` 旁。focus 有 `focusPool` 且 blocks 无覆盖证据 → 注入最优 place 为 `attraction`（`reason: hard_must_include`），再按 pace 裁软块。不靠 LLM 听 prompt。
5. **硬失败：** `applyMustIncludeDayEvidence` 后，agent 路径若本日本应覆盖的 focus **仍 missing** → 抛结构化错误（禁止软成功）。末日仍有任意 missing → 维持 `present_day_then_cover_must_include`（禁总览）。
6. **不做：** 「必去=无」不自动发明一日游；不改 Xi’an hard must-see 语义；host（2play Mode H）路径不硬抛（返回 missing，由 2play 自处）。

**P0 验收（硬闸）：** 必去 **辛特拉、卡斯凯什**（4 天）→ 两天 `blocks` 各有可证据覆盖的真实地点（pool name/`native_id` 或 10km+name）；第 2 天调用省略 `must_include` 仍 focus 未覆盖项（sticky）；LLM 只排贝伦、focusPool 有辛特拉点 → 注入后 covered；focus 搜空 → `ok:false`；必去 **无** → 不发明一日游。

**P1 — 末日卡单次展示（P0 绿之后）：**

1. [`arrange-present-gate.ts`](../../1.places-agent/src/mcp/arrange-present-gate.ts) 渲染 `presentation.user_visible_markdown`（日卡；末日附 overview 一天一行）。
2. `host_instructions` 改为「**只粘贴 `presentation.user_visible_markdown` 一次后 STOP**」，禁止宿主自行再拼日卡。
3. 会话 `overview_emitted` / `trip_complete`：末日成功后置位；同 key 再调 `arrange_day` → `{ ok:true, already_complete:true, next_action:"stop", presentation }`。
4. 同轮双贴仍可能，但把唯一真相收成一个字符串；ChatBox 侧最终应用 blob 幂等（本仓外）。

**D9 follow-up（2026-08-23，根因复盘后）：**

1. **`covered` 跨日 sticky 复核：** `applyMustIncludeDayEvidence` 本就会把命中 token push 进会话 `covered` 并 `sessionByKey.set` 持久化，`getAssignedMustIncludeTokenForDay` / `selectMustIncludeFocusToken` 均跳过已 covered。样本里「Day 4 仍 missing 辛特拉」来自 pre-D9-restart 旧 build。新增多日 sticky + 不重排单测锁死该行为。
2. **内部 reason 不外泄：** `buildDayCardMarkdown` 原把 `b.reason` 原样贴出，导致 `hard_must_include` / `hard_must_see` 等保护性 trim 标记直出给用户。改为对内部 marker 集合跳过该行（保留 header），非 marker reason 照常渲染。
3. **未做：** `navigate` deeplink `query=0,0` 属宿主/上游 geocode 侧问题，不在 D9 服务端范围内，单列跟进。

### D4 — L1 must-see / 热门（关闭 ADR-042 / ADR-040 D8）

1. **禁止**扩 `discover-must-see` CATALOG。  
2. **目的地无关**信号：通用热门/必去 **模板** query（非城百科专名表）+ Google `searchText` **`rankPreference: RELEVANCE`**（API 仅允许 `RELEVANCE` | `DISTANCE`）。**禁止**传 `POPULARITY`（Google 400 `INVALID_ARGUMENT`，会清空景点池、只剩餐厅——Lisbon ChatBox 已复现）。  
3. 冻结城表仅作临时 boost，不得当交付。  
4. 验收：西安 + **Lisbon** 池头含约定地标类覆盖（探针 token），不得靠加城行。  
5. **ChatBox 展示：** 每日上屏须富文本日卡；最后一天后须 **多日总览**。Intake = **聊天侧收齐再调工具**（Option A）；缺字段安全网返回整卷 `questions[]`。**不能**依赖 Cursor `AskUserQuestions` UI。

### D5 — 交通裁定（废止「待决」）

- 算路：**Google / AMAP Directions**（ADR-022 / Feature 37）。  
- 字段：`legs_to_here[]`；可选 `from_origin` / `to_destination`；失败进 `skipped` / `transit_outcome`。  
- ChatBox 必须走 agent 才自带腿；2play 走 enrich HTTP（as-built 已接线）。

### D6 — As-built / target（ADR-039）

| 能力 | Producer (agent) | Consumer (2play) | ChatBox MCP |
| --- | --- | --- | --- |
| Mode H host | Done | Done（`/api/plan`） | **禁止作默认**（本 ADR D2） |
| Feature 37 legs | Done | Done（enrich） | 随 agent 响应 |
| 城表 must-see | 冻结债 | 不依赖 | 不依赖 |
| 热门 L1 | **本包 target→落地** | 自动受益 | 自动受益 |
| MCP 强制 agent / 软闸 | **本包落地** | N/A | 依赖 |
| must_include 硬覆盖 D7 | **本包落地（共用 arrangeDay）** | 同字段受益 | 同字段 + 续排文案 |
| 空候选自动 discover D8 | **本包落地（共用 arrangeDay）** | 同路径 | 空数组可排；错误文案纠偏 |
| must_include 强制排日 D9 P0 | **本包落地（共用 arrangeDay）** | 同字段受益 | 关 open issue #2 |
| 末日卡单次展示 D9 P1 | **本包落地（MCP 层）** | N/A | presentation blob |

## Rationale

- Lisbon 失败主因是 **宿主走 host + 无 Lisbon 热门**，不是 2play 未做 enrich。  
- 强制 MCP agent 比改客户端设定更符合产品约束。  
- 热门模板 query + 合法 RankPreference（RELEVANCE）满足 ADR-042，避免第三次城表；误用 POPULARITY 比「无模板」更糟（整池 400）。  
- 必去假覆盖的根因是 **主题字符串闸 + 错误候选**；必须在共用核心自动搜并硬校验，且 HTTP/MCP 一致（ADR-018）。  
- 空候选硬失败与产品「仅 city/date/days 硬必填」冲突；补池必须在共用核心，不能靠宿主 search。

## Consequences

- 实现：MCP create-server 强制 agent；arrange 软闸；Google RELEVANCE（禁 POPULARITY）；discover 模板热门；富展示 + 总览 host_instructions；**D7 `arrangeDay` 自动搜+硬覆盖**；**D8 空候选自动 discover + 错误/总览文案**；双仓 specs / performance / knowledge 纠偏。  
- 测：MCP host→仍 agent；intake；软闸；legs；Lisbon/西安池头；CATALOG 冻结；Google 不发送 POPULARITY；**HTTP/MCP `must_include_coverage`（假辛特拉/贝伦冒充仍 missing；半径内 covered）**；**空 `candidates` + city → 自动 discover 后可排日**；工具 description 含城市+天数引导。  
- 旧 ADR：037/038/040/042 以本 ADR 为关闭包指针。

### D9 精简（2026-08-23，过度设计复盘后）

D9 P0/P1 落地后样本与架构复盘发现三类过度设计：P1 展示死代码（宿主忽略 `user_visible_markdown`/`overview_emitted`，Day 4 不重复靠的是 `host_instructions` 文本控制句）、`must_include` 过度机制（assignment 分日表 + 确定性注入制造低质块并外泄内部 reason）、城市硬编码（must-see 簇名单等五处，违反 ADR-042 原则）。本节记录精简，详见 [ADR-042 Update](./ADR-042-no-city-encyclopedia-in-source.md)。

1. **删 P1 展示死代码：** 删 `arrange-present-gate.ts` 的 `overviewEmittedSet`/`isOverviewEmitted`/`markOverviewEmitted`/`buildDayCardMarkdown`/`buildArrangePresentationBlob`/`INTERNAL_REASON_MARKERS` 及 `create-server.ts` 的 `presentation` 字段与 `already_complete` 分支。保留 `next_action` 枚举与 `host_instructions` 控制句（「Do NOT call arrange_day again / Do not invent Day N+1」）——宿主实际遵守的是这些。
2. **assignment 降级 + theme 门控 focus：** 删 `seedMustIncludeAssignment`/`getAssignedMustIncludeTokenForDay`/`SessionEntry.assignment`。`selectMustIncludeFocusToken` 改为 **theme 门控**：仅当 `day_theme` 命名某个 missing token 时才强制 focus；无 theme 或 theme 不匹配 → 返回 null（不强制），让 themed 那天来认领（如 Day 1 无 theme → 排老城，Day 2 `day_theme=辛特拉一日` → 排辛特拉全天）。P0 覆盖仍由末日闸兜底保证。保留 `covered` 跨日 sticky。
3. **注入改硬失败重试：** 删 `ensureHardMustIncludeCoverage` 注入。`applyMustIncludeDayEvidence` 后若本日 focus token 仍 missing → 抛结构化错误，由 `callItineraryLlmWithValidationRetry` 带「Previous errors: must_include X still missing」重试一次；仍失败 → `ok:false`。不再制造低质注入块、不再外泄 `hard_must_include` reason。
4. **删五处城市硬编码：** 见 [ADR-042 Update](./ADR-042-no-city-encyclopedia-in-source.md) 五处表。回退到与目的地无关通用逻辑。
5. **必去知识搬家（方案 2）：** 新增 `discover-must-see-llm.ts` `inferMustSeeFromPool`，discover 后 LLM 从候选池选 ≤3 公认必去（prompt 无城市名），经池校验防幻觉，作为 `must_include` 种子走现有 P0 硬通道。对所有城市通用。过渡方案，热度包落地后可下线。
6. **新增守卫测试：** `tests/no-city-hardcode.test.ts` grep 扫源码断言无城市 POI 知识字串，把 ADR-042 原则钉成 CI 闸。
7. **全天 day-trip prompt：** `buildSchedulePrompt` 的 `HARD MUST SCHEDULE` 原文「you may mix with other city candidates for a half-day」鼓励 LLM 把 day-trip 小镇排成半天。改为「这些地点同处一个优先目的地，作为当日主焦点；若是远离基市的 day-trip 小镇，dedicate the FULL day，不要掺远郊基市点（旅行者不可能半天辛特拉/卡斯凯什再回城观光）」。
8. **续排措辞（实验后定稿 step2）：** `host_instructions` 要求「ONE day at a time：呈现 Day N 卡后自行调 Day N+1，不等用户、不并行」。实验记录：①「IMMEDIATELY next day」（原版）→ 宿主并发 4 次再倾倒；②「in a SEPARATE turn」→ 宿主误读为「等用户下一回合」，问「如果你愿意」；③「SAME response / never 如果你愿意」→ 回退到并发。最终定稿 step2「ONE day at a time, no asking」。**教训：措辞无法强制宿主 LLM 行为（问确认、并发）——宿主系统提示优先于 host_instructions；服务端唯一硬杠杆是软闸，但对同回合并发调用有竞态（no-op），且挡不住「问完再发」。续排顺序最终依赖宿主遵守，无服务端硬保证。

**精简后保留的核心（不过度）：** 真实 POI 搜索、交通 ETA（Directions + heuristic fallback）、LLM 选点 + 校验 + 重试、AbortSignal、legacy fallback、`must_include` P0 硬失败、`covered` sticky。

## Date
2026-08-23
