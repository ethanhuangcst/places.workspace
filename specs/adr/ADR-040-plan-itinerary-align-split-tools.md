# ADR-040: plan_itinerary 对齐拆分工具 + MCP 编排

## Status
Accepted（**关闭包指针：** [ADR-043](./ADR-043-chatbox-mcp-and-cross-product-closure.md) — 2play Mode H+enrich as-built；ChatBox MCP 强制 agent；L1 POPULARITY）

## Context
[ADR-032](./ADR-032-llm-itinerary-mcp-tool-split.md) 写明 HTTP `plan_itinerary`「内部 discover + arrange × N」，但 as-built 是 `searchCandidates` + **一次多日 LLM**（`llmPlanItinerary`），与公开工具 `discover_places` / `arrange_day` 并行。Mode H 下 MCP `arrange_day` execution=host 实测工具侧常 &lt;10s；2play 主路径已不依赖 `plan_itinerary`（[ADR-037](./ADR-037-where2play-plan-l2-quanzil.md)）。需决定一站式是否收敛到拆分路径、候选上限，以及 ChatBox MCP 侧如何先问清条件、按天上屏。

状态记录约定见 [ADR-039](./ADR-039-cross-product-as-built-vs-target.md)。

## Decision（逐条）

### D1 — L1 与候选上限（**已同意 2026-08-23**）

1. `plan_itinerary`（LLM 路径）Phase 1 **改为调用** `discoverPlaces`（不再经旁路 `searchCandidates` 直接碰池）。  
2. 交付拆成 **两个用户故事**（分步验收）：  
   - **Story A：** 仅接线 `discoverPlaces`，截断仍为现网 `8 × min(numDays, 3)`。  
   - **Story B：** 将 discover 池与进 prompt 的上限统一放宽为 **16**（`CANDIDATE_CAP` 与 `buildUserMessage` 的 `MAX_CANDIDATES`；与 2play `ARRANGE_PROMPT_CANDIDATE_LIMIT = 16` 对齐）。  
3. 不在同一故事里混做 L2 改造。  
4. **L1 质量不另开旁路：** `discoverPlaces` / 壳内 L1 **必须**走 [ADR-038](./ADR-038-discover-places-quality.md)（热门城 must-see seed query + 过滤/排序）。仅「换成调用 `discoverPlaces`」**不等于**完成本批 L1；见 **D8**。

### D2 — `plan_itinerary` 的 L2 改为 `arrange_day` × N（**已同意 2026-08-23 · 选 A**）

1. HTTP/MCP `plan_itinerary`（LLM 路径）在 Phase 1 使用 `discoverPlaces` 之后，L2 **改为**对 dayIndex=1…N 调用 **`arrangeDay`**（传入收缩后的候选与 `exclude_names`），再拼成完整行程返回。  
2. **硬约束（已确认）：禁止** `plan_itinerary` **一次**在 places-agent 侧用 **单次 places-agent LLM 调用**把 **多日行程全部**生成完（即废止主路径 `llmPlanItinerary`「整单一次多日 JSON」）。  
3. **允许**的一站式实现：内部 **按天**调用 `arrange_day`（首版默认每日本地 `execution=agent`），再在服务端拼接；这是 N 次单日 LLM，不是「一次多日 LLM」。legacy `ITINERARY_MODE=legacy` 仅作显式降级，不作产品主路径。  
4. 与公开工具同源：硬必去、真交通等与单独调 `arrange_day` 一致。  
5. 交付：在 D1 Story A/B **之后**单独用户故事（Story C：L2 真壳）；勿与 A/B 混验。  
6. ChatBox 仍按 **D3** 走拆分+按天上屏；一站式壳化服务脚本/旧客户端/要整包 JSON 的调用方。

### D3 — MCP（ChatBox）交互编排（**已同意 2026-08-23**；**先抽取后追问** 同日加固；**2026-08-23 补丁：不依赖客户端 system prompt**）

1. **先抽取、缺了再问（硬规则，适用于 Q1–Q11 每一题）：**  
   - 宿主必须先从 **当前用户消息 + 本轮对话已有内容** 抽取可映射字段；**读得到的题一律跳过，不得复问**。  
   - **仅当**该字段在输入中 **读不到**（且无合理默认可静默采用）时，才按题序 **一次只问一题**。  
   - **禁止**为「走完问卷」而明知故问。  
2. **不问「天气」。**  
3. **目的地 / 「城市」字段：** 地点永为 Q1；参数名仍为 `city`。  
4. **默认编排：** `discover_places` → 按天 `arrange_day` → **当日结果先写入对话** → 再排下一天。  
5. **落地（修订）：以工具契约为主，不强制终端用户改 ChatBox system prompt。**  
   - **服务端 intake：** `discover_places` / `arrange_day` 在缺必填边界时返回 **单条** `intake.question` + `host_instructions`（一次一问）。  
   - MCP tool description 继续承载 D3/D7 路由。  
   - knowledge 中的 system prompt 模板仅为 **可选运维增强**，不得作为产品验收前提。  
6. 条件映射表（Q1–Q11）仍有效；人数见 D6。

### D4 — MCP 默认 `execution=agent`（**2026-08-23 修订**；原 host 默认废止为产品默认）

1. **MCP** `arrange_day`：省略 `execution` 时默认 **`agent`**（服务端返回带 `start_time` / 交通腿的当日 JSON）。显式 `execution=host` 仍可用于 Mode H handoff。  
2. **HTTP** `/v1/arrange_day`：默认保持 **`agent`**（不变）。  
3. D2 一站式壳内部仍 **固定 `execution=agent`**。  
4. **修订理由：** 不得要求终端用户修改 ChatBox system prompt；host 默认依赖宿主自觉执行 prompt，实测会批量调工具并丢掉时刻/交通。agent 默认把结构化当日结果交回宿主展示。  
5. 测：MCP 不传 `execution` → agent 路径（非 host handoff）。

### D5 — 保留 `plan_itinerary` 为真壳汇聚门面；MCP 软降级（**已同意 2026-08-23**）

1. **保留** HTTP + MCP 注册：作为「已收齐的行程边界 → 完整行程 JSON」一键入口。  
2. **必须真壳（与 D2 合一）：** 调用 `plan_itinerary` = 自动 `discoverPlaces` + `arrangeDay`×N，与显式调用同源（质量/硬必去/真交通一致）。  
3. **同源 ≠ 同体验：** ChatBox 主路径仍 **D3**（按天上屏）；一站式工具仍一次返回整包 JSON（用户常仍要等拼完）。ADR 写的「一样」指编排与质量，不是交互等同 D3。  
4. **MCP 软降级（强化）：** tool description **不得**把「推荐行程 / N日游」标为 `plan_itinerary` 触发语；缺 bounds/origin 时返回 Option A `need_input` 并 `prefer_tool: discover_places`。**禁止**缺日期时默认「今天」只排 1 天再让宿主散文补后几天。  
5. **不**在本阶段从 MCP 硬删或下线 HTTP。

### D6 — 人数（`party_size`）与 2play 对齐（**已同意 2026-08-23 · 选 C**；D3 **Q4 = 人数**，起点前）

1. agent：`arrange_day` 与 `plan_itinerary`（壳透传）增加可选 **`party_size`**：整数 **1–20**；省略 = 不暗示人数。  
2. **不**加到 `discover_places`（L1 不依赖人数）。  
3. arrange prompt / Mode H user 文案注入一行，例如 `party_size: N`；指导模型：N≥6 偏大桌/团体节奏；婴幼儿等细节仍靠 Q11（不做硬规则引擎）。  
4. **D3 提问：** 地点 **永为 Q1**；人数为 **Q4**（「一共几人出行？」），位于起点（Q5）之前；输入已含人数则跳过；否则默认 **2**。  
5. 2play：OPENAI_CN constraints **写入** `partySize`；若走 agent arrange / 壳，映射 `party_size`。  
6. **明确不做（本条）：** 儿童/成人拆分、无障碍独立枚举（继续 Q11）；`timeFrom`/`timeTo` 已有路径，不另开 MCP 必问题。

### D7 — 多 MCP 并存下提高 places-agent 命中率（**2026-08-23**）

**硬约束：** **不得**要求用户「只开 places-agent / 关掉其它 MCP / 必须加前缀」。多 MCP 同开是常态。

| # | 杠杆 | 状态 |
| --- | --- | --- |
| **1** | MCP tool description / 触发语 | **已同意** |
| **2** | 保留 `plan_itinerary` 名（D5） | **已同意** |
| **3** | 宿主 system prompt（ChatBox/Cursor） | **可选运维增强 only** — **禁止**作为终端用户必做步骤（2026-08-23） |
| **4** | description 首句 / server 名 | **已同意**（与 #1 同批） |
| **5** | `plan_itinerary` **别名**（`trip_plan`、`trips`） | **已同意**；**别名转调，禁止复制实现** |

#### #3 — 可选运维模板（非用户必做）

places-agent **不要求** ChatBox 用户改 system prompt。产品路径靠 **工具 intake + description + MCP 默认 agent**。  
仓库 knowledge 可保留可复制模板，仅供自建助手的运营者选用。

#### #4（已同意）

保持 `serverInfo.name = places-agent`；行程相关工具 **description 第一句**含中英触发语。与 #1 同 Story。

#### #5（已同意）— 别名，不复制代码

注册 MCP（及可选 HTTP）工具名 **`trip_plan`、`trips`**，handler **仅转发**到与 `plan_itinerary` **同一函数**（真壳 D2：`discoverPlaces` + `arrangeDay`×N）。

- **允许：** `registerTool("trip_plan", …, (args) => sameHandler(args))` 或薄包装一层。  
- **禁止：** 复制 `planItinerary` / LLM 编排第二份实现。  
- description：名称磁铁 + 标明「alias of plan_itinerary；非聊天默认；偏好 D3 按天」。  
- 测：别名与 `plan_itinerary` 契约/行为一致（同一入口断言即可）。

**落地 Story 摘要：** description（#1+#4）+ 别名注册（#5）+ guide 模板（#3 可选复制）+ 去掉「只开 places」强制表述；验收在多 MCP 同开下抽样。

### D8 — 本批强制纳入 L1 must-see/**热门**能力（**2026-08-23 补入**；机制受 [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) 约束）

**背景：** [ADR-038](./ADR-038-discover-places-quality.md) 曾用城表 CATALOG 止血；代码有 `discover-must-see`，但非表内城（如 Lisbon）无 must-see。本批原 D1–D7 未把 L1 质量当硬门禁。**不得**再用「往 CATALOG 加城」关闭本条（ADR-042）。

**决策：**

1. **本批 DoD：** HTTP/MCP `discover_places`（及壳内 L1）须具备**目的地可扩展的**热门/必去信号路径（供应商热门或外置 pack 等，见 ADR-042）；至少验证：**一表内锚点（如西安）+ 一非表内目的地（如 Lisbon）** 池头不是「仅靠泛搜碰运气」。  
2. **按 ADR-039：** 现有 CATALOG 接线 = 临时 as-built，**不是**本批 target 完成态。  
3. **与 D3：** intake 不替代 L1 热门能力。  
4. **测：** 禁止仅用 `mustSeeQueriesForCity("西安")` 当产品完成证明。

## Rationale

- **D1：** L1 卫生对齐 + 16 与 2play 一致；分故事验收；**L1 质量绑定 ADR-038（D8）**。  
- **D2（A）：** 真壳按天 arrange；**禁止**「单次 agent LLM 生成全部多日」；壳内可按天 `execution=agent` 再拼接。  
- **D3：** **先抽取、缺了再问**；地点永为 Q1；人数在起点前（Q4）；按天上屏；**以工具 intake 落地，不强制改客户端 system prompt**。  
- **D4'：** MCP 默认 **agent**（2026-08-23 修订）；HTTP arrange 默认仍 agent；显式 `execution=host` 保留 Mode H。  
- **D5：** 保留真壳汇聚；同源质量 ≠ 同体验；MCP 软降级。  
- **D6（C）：** `party_size` 一等字段；D3 Q4 收集；2play / agent L2 真正消费。  
- **D7：** #1/#2/#4/#5 已同意（别名转调、不复制）；#3 仅可选运维模板，**禁止**作为终端用户必做。  
- **D8：** 本批硬纳入 L1 热门/must-see **能力**（ADR-042：禁扩城表）；须非表内目的地亦过线。

## Consequences

- D1：agent-stories Story A/B；测 plan L1≡discover；Story B 更新 cap 断言。  
- D2：Story C — `planItinerary` 编排 `arrangeDay`×N；删除或降级主路径对 `llmPlanItinerary` 的依赖；回归 E2E/guide；更新 ADR-032 Consequences（as-built = 真壳）。  
- D3：MCP/HTTP 缺字段返回单条 `intake` + `host_instructions`；MCP description；题序以本表为准。  
- D4'：MCP omit `execution` → agent；测与 guide；Mode H 仍可显式 host。  
- D5：guide/description 软降级；E2E 保留 HTTP `plan_itinerary`；实现与 D2 Story C 同批或紧随。  
- D6：Story D（agent `party_size` + prompt + 测）→ Story E（2play OPENAI_CN + agent body 映射）；D3 文案含 Q4 人数；**排在 A→B→C 之后**，可与 D3 文案并行设计。  
- D7：description（#1+#4）+ `trip_plan`/`trips` 别名转调（#5）+ 可选 ops prompt 模板（#3，非用户必做）；验收多 MCP 同开。  
- **D8：** 本批关闭门 = 目的地可扩展的热门/must-see 路径（ADR-042）；西安+非表内城（如 Lisbon）验证；禁止加 CATALOG 行充数。  
- 建议故事顺序：**A → B → C**；**D3+D4'+D5+D7** 可与 A 并行；**D6** 在 C 后；**D8 与 A/B 同批，实现须换机制而非扩表**。

## Date
2026-08-23
