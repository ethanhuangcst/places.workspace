## 2026-09-09 — MVP-T1 收尾：设计合并进 agent-design

- **范围：** `mvp-1t-closing-plan.md`、`agent-design.md`、链接纠偏、冗余稿删/stub。
- **变更：** Item 1 usable Confirmed；`tmp-0909` + `real-agent-refactory` 真源并入 `agent-design`（真智能体能力/原则 + MVP-T1 as-built：逐题 session、芯片 collected 可空、起飞不强制 L3）；去重 T1 节；ADR-050 / architecture / backlog / stories 指针改锚 `agent-design`；`tmp-0909` 删除；`real-agent-refactory` / `draft-nominate-must-see-prompt` 改为短指针。
- **关联：** ADR-050、ADR-060、`agent-itinerary-93a` / `96`。

## 2026-09-09 — 出行限制 4 列网格

- **范围：** `plan-constraints` mockup + where2play；`2play-plan-90a` AC12。
- **变更：** 只读约束条改为 4 列上标下值：起飞 8 项两行（类型在预算前），横线下 intake 一行四格；必去点 chip。产品与 `06-plan-qa` 对齐。
- **关联：** `plan-constraints-panel.tsx`、`plan-intake.ts`、`ui-mockup/06-plan-qa.html`。

## 2026-09-09 — tmp-0909：起飞 2play↔agent 时序与数据流

- **范围：** `specs/agent-specs/tmp-0909.md`。
- **变更：** 按 ADR-060 重写：起飞 `plan_trip` intake 方法链、时序图、数据流图、后 4 问交互、与 L3/全环边界；去掉过时「强制提名 / 提示词 A/B/C」叙述。
- **关联：** ADR-060、`agent-itinerary-96`。

## 2026-09-09 — intake 去掉强制提名（纯环出芯片）

- **范围：** `plan_trip` intake、`agent-itinerary-96`、2play Q3、ADR-060。
- **变更：** 删除 `runForcedNominate`；芯片仅环内 `search_places`/`commit_trip`；同 trip 复用已有 must_see；Q3 空态 i18n + 手输 + 再 fetch。95 AC1 superseded。
- **关联：** [ADR-060](./adr/ADR-060-intake-no-forced-nominate.md)、`tmp-0909.md`。

## 2026-09-09 — L3 提名提示词收紧 + 落地跨语种 token 匹配

- **范围：** `buildNominateMustSeeUserMessage`（CN/HK/TW/EN）、`pickNominatedGroundCard`、`groundNominatedNameOnce` suggest 候选过滤。
- **变更：** 近郊改「目的地近郊有可安排一日游/半日游的知名必去地时，应至少列入一个」（时间预算+质量门槛+保底一个）；落地加 `sharedProperToken` 回退——子串不命中时按共享专名 token（≥4 字符、去通用场馆词）匹配，修 `Mosteiro dos Jerónimos`↔`Jerónimos Monastery` 跨语种别名。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`、ADR-042。

## 2026-09-09 — L3 提名提示词收紧（近郊/地理多样/非点）

- **范围：** `buildNominateMustSeeUserMessage`（CN/HK/TW/EN）。
- **变更：** 近郊由「也可」改「应至少列一个」；多样由「不要同一类」改「不要全集中在同一片区域」；加「不要线路/活动/打包描述」。措辞收紧，不加长。探针杭州 FAIL→PASS（近郊芯片出现）。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去门诚实化 + 一日游宽搜 + 拒绝规则合并

- **范围：** 探针近郊门、`groundNominatedName`、`pickNominatedGroundCard`、`nominateMustSeeViaLlm`。
- **变更：** 近郊门改为距锚点 >20km（不再用簇键启发式；20km 为近郊/一日游门槛，Sintra 23km 算）；落地加去后缀宽搜回退（>15km）；拒绝规则收成 `isNoiseCategory` + `isVagueAreaName`；suggest 有 tip 才重试；同 `native_id` 去重；探针 `attemptable` 分母对齐 `isVagueAreaName`。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去落地：suggest 优先 + 短名芯片

- **范围：** `groundNominatedName`、`mustSeeChipLabel`、`capClusterOccupancy`、双城探针。
- **变更：** 提名先 `suggest_places` 再同名 `search_places`；芯片 label 用提名短名；同簇最多 3；探针门加垃圾店/同簇/近郊。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去提名 L3：JSON 短名 + 季节词

- **范围：** `buildNominateMustSeeUserMessage`、`parseNominatePlaceNames`、`pickNominatedGroundCard`。
- **变更：** 只输出 JSON 短地名；有 `bounds` 时拼月份+季节；落地跳过住宿/购物/整湖泛名。探针杭州 AMAP + 里斯本 GMAP 过门。
- **关联：** `tmp-0909.md`、ADR-059、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — ADR-059 规划必须传入已知条件

- **范围：** 必去提名 L3；`plan_trip` / `discover_places` 转发 `bounds` 等。
- **变更：** 已有日期、起点名、用户必去、其他要求写入提名参数行；禁止因「只要天数」丢掉日历。不写城×季节表。
- **关联：** [ADR-059](./adr/ADR-059-pass-all-known-trip-constraints.md)、`places-ontology.ts`、`agent-itinerary-95` AC1。

## 2026-09-09 — 起点补全产品验收通过

- **范围：** `2play-plan-96` AC5。
- **变更：** 杭州 + `SFEE` 产品路径用户确认通过。
- **关联：** `knowledge/maps/origin-autocomplete-amap-gmap-probe.md`。

## 2026-09-09 — 起点：suggest_places 优先再 search

- **范围：** 起点验真（`2play-plan-96` AC5 / Story 5 AC16）、agent `suggest_places`、ADR-053 查找路径。
- **变更：** 一律先厂商 autocomplete（AMAP inputtips / Google places:autocomplete），按目的地收窄后再 hydrate/`search_places`；提示空才搜点。不分字种/字母长度。探针：`amap-sfee-origin-probe`、`origin-autocomplete-amap-gmap-probe`。
- **关联：** `plan-resolve-origin.ts`、`places-agent` adapters + `/v1/suggest_places`、ADR-053 D1/D4。

## 2026-09-09 — 起点验真线程：部分/零匹配文案

- **范围：** `2play-plan-96`、助手 hotel 题。
- **变更：** 精确匹配进下一题；部分匹配只显示 candidates 文案 + 纵向芯片；零匹配只显示 not_found；二者都不显示 `need_prompt.hotel`；再搜先清上一轮候选。
- **关联：** `2play-design.md` §4.12 Story 5。

## 2026-09-09 — 助手过程芯片进线程 + 约束条 mockup

- **范围：** 助手 1–3；规划栏仅 spec mockup。
- **变更：** 住宿/必去芯片进对话线程；dock 只留跳过/重答；去掉底部跳过提示；左上拉手外拉变大。`06-plan-qa` 约束条改为 4 列 + 宽格。
- **关联：** `plan-assistant-nav.tsx`、`plan-nav-resize.ts`、`2play-plan-97` AC5。

## 2026-09-09 — MVP-T1 故事 AC 签收

- **范围：** `93a` / `90a` / `96` / `97` / `98` / `99`。
- **变更：** 对照测试与代码签收为 Done；补 `2play-plan-99` GWT；`90a` AC2 写清 T1 不二次 `plan_trip`。产品 usable 仍待人确认。
- **关联：** `agent-stories.md`、`2play-stories.md`、`product-backlog.md`、`plan.md`。

## 2026-09-08 — Debug 页列出地图 source

- **范围：** `2play-plan-98` debug 表。
- **变更：** 候选/registry 从 card `provider` 或 `sources[]` 取供应商；`list_destination_pois` 回传 provider；debug 用 `play.plan.provider.*`；增加 Origin/stay 行。
- **关联：** `plan-discover-pool.ts`、`dispatch.ts` list_destination_pois。

## 2026-09-08 — 约束条对齐 8 字段起飞 + intake 回写

- **范围：** `plan-constraints` / `2play-plan-90a` AC12。
- **变更：** 字段序改回 mockup（类型/节奏/交通在住宿前）；节奏/交通用目录 i18n；agent 四问写入约束条。
- **关联：** `ui-mockup/06-plan-qa.html`。

## 2026-09-08 — 必去芯片：服务闸 + 8 条 + 排版

- **范围：** `agent-itinerary-95` AC2/AC6、`2play-plan-97` AC5。
- **变更：** L0 丢停靠点/游船/码头等服务类卫星 POI；L2「可停靠」改为「可落到地图上的一个点」；`MUST_SEE_LIMIT` 5→8；必去芯片纵向一行一点；跳过/重答钉在输入框上方左侧。
- **关联：** ADR-042（目的地无关模板，不扩 CATALOG）。

## 2026-09-08 — L0–L3 必去合同落地

- **范围：** `agent-itinerary-95` / 95b / 95c、`2play-plan-99`；稿 `draft-nominate-must-see-prompt.md` Accepted。
- **变更：** 设计并入 L0 无坐标丢弃、L1 工具一句、L2 系统本体、L3 四语可落点短句（近郊不限一处）；字段表不进提名 user；稳定 key 传 agent；同一 trip 换 locale 不重提名。
- **关联：** ADR-042、ADR-050。

## 2026-09-08 — 必去提名纳入行程类型 + 10 城审核

- **范围：** `buildNominateMustSeeUserMessage`；审核稿 `nominate-must-see-10city-audit.md`。
- **变更：** 提名提示使用已收集 `trip_type`/pace/budget；合适时含一日游；无城市 POI 表。10 城 × 中英直连名单供人审。
- **关联：** ADR-042、`agent-itinerary-95`。

## 2026-09-08 — T1 质量：提名去重、起点满匹配、助手导航、stop 源

- **范围：** `agent-itinerary-95`、`2play-plan-96`、`2play-plan-97`、`2play-plan-98`（计划稿曾用 93b/90b/90c/91，与 T2/T3 id 冲突故改号）。
- **变更：** backlog/stories/design 写入 GWT；intake 强制 LLM 提名+落地+族去重；起点仅满匹配唯一才 auto-hit；重答/跳过快答；debug 与行程标 Google/高德。
- **关联：** ADR-042、ADR-050、ADR-053。

## 2026-09-07 — T1：plan_trip 写 session、debug 池、registry 芯片、酒店跳过/验真、不自动 fill

- **范围：** `2play-plan-90a` T1 切片（不含 T2 骨架密度）。
- **变更：** `POST /api/plan/trip` 成功后 upsert `planSessionCache`；debug 页在 focus/轮询下重拉 current 与 stops-pool；fixture/空搜时 `must_see` 芯片取自城市 AttractionPoi；酒店空=skip、非空 422 不吞；四问后不再 `runPlan` NDJSON。
- **关联：** `2play-plan-90a`、`where2play/app/api/plan/trip/route.ts`、`places-agent/src/core/plan-trip.ts`。

## 2026-09-07 — 真智能体合规：芯片权威 + 省略 providers + 芯片走 plan_trip

- **范围：** must_see 由 LLM 提名落地卡标记；2play 产品路径省略 `providers[]`；discover 改 `plan_trip` + `fetch_trip_details`。
- **变更：**
  - `applyNominatedMustSee`：芯片 = 落地提名；heat 只补缺与同名平手。
  - `startPlanDiscover` 调 `plan_trip`，不再 `discover_places`。
  - `/api/plan`、discover、must-see-suggestions、travel_tips 不再拼 `providers[]`。
  - 前缀族去重仅作搜噪闸；去掉会误并「纪念馆」的后缀。
  - `pickLodgingStayCard` 与 2play 对齐：山庄等关键词 + 唯一 token 命中。
- **关联：** ADR-042、ADR-050、ADR-052、real-agent-refactory G6/G7。

## 2026-09-07 — 地图路由质量修复（大陆禁 Google 回退 + drive_preferred + 交通 UI 结构化 + LLM 必去提名 + 前缀族去重）

- **范围：** places-agent 与 where2play 地图路由质量问题修复，覆盖起点验真、必去点质量、排餐供应商、交通引擎、交通 UI。
- **变更：**
  - **起点验真**：`looksLikeLodging` 扩住宿关键词（山庄/公寓/民宿/别墅/hostel/bnb 等）；`ORIGIN_CANDIDATE_LIMIT` 3→6；新增 token-priority 匹配（用户 query token 唯一覆盖候选时直接命中，绕过 category 判定）。
  - **大陆禁 Google 回退**：`shouldTryGoogleAfterEmptyAmap` 无条件返回 `false`（景点/餐厅/交通）。
  - **排餐**：AMAP 餐厅搜索半径 `DINING_AROUND_RADIUS_M` 1000m→3000m。
  - **交通引擎**：`ItineraryPreferences` 新增 `drive_preferred`；`transitPreferred` 正则补 "交通"；AMAP Directions 调用传 `city` 参数（修复非上海城市默认上海）；坐标匹配优先 `native_id` → 精确名 → 归一化名；失败标记 `transit_outcome: "partial"`。
  - **必去点**：`nominateMustSeeViaLlm` LLM 提名 3-5 必去点（含 day-trip）→ `search_places` 落地坐标。
  - **前缀族去重**：`attractionClusterKey` 剥离 "景区"/"售票处"/"重建记" 等卫星后缀，避免同一地标（如雷峰塔）重复推荐。
  - **交通 UI**：`ItineraryTransitSlot` 扩展 `from` / `to` / `legs[]` / `outcome`；行程详情页结构化渲染交通药丸（transit-from / transit-to / transit-option）。
- **关联：** [ADR-052](./adr/ADR-052-map-provider-routing.md)（D4/D7/Consequences 修订）；`plan-resolve-origin.ts`、`tools.ts`、`amap/direct.ts`、`itinerary-timed.ts`、`plan-agent-body.ts`、`plan-enrich-transit.ts`、`itinerary-planner.ts`、`discover-dedupe.ts`、`itinerary-types.ts`、`itinerary-skeleton-map.ts`、`plan-skeleton-fill.ts`、`plan-itinerary-view.tsx`。

## 2026-09-07 — MVP-T1 开工（need_input 4 题 + 大陆 seed + 开关机制）

- **范围：** `agent-itinerary-93a` 问题集；测试策略写清 seed/真 API 开关；大陆 4 城 stops pool。
- **变更：**
  - `defaultNeedInput`：`hotel` / `start_time` / `must_see` / `other`；`ask_user` 保留 options；collected 回填 must_see 芯片。
  - [`agent-test-plan.md`](./agent-specs/agent-test-plan.md) 增 §1.3 开关机制 + MVP-T1 用例。
  - [`seed-city-pois.ts`](../places-agent/scripts/seed-city-pois.ts) 加杭州/西安/上海/厦门；大陆 AMAP 文本分页。
  - Probe 文件名加 sha1，避免 CJK geocode 串城（见 knowledge `probe-cache-cjk-filename.md`）。
  - UI mock：[`06-plan.html`](./2play-specs/ui-mockup/06-plan.html) 8 字段；[`06-plan-qa.html`](./2play-specs/ui-mockup/06-plan-qa.html) 4 问逐题。
- **关联：** ADR-052、ADR-057、ADR-058；`plan-trip.ts`。

## 2026-09-07 — POC 签收（agent-poc-01 Done）

- **范围：** 真智能体 POC 标 Done；工作计划切到 MVP-T1。
- **变更：**
  - [`product-backlog.md`](./product-backlog.md) §0 / §1：`agent-poc-01` → Done；下一步 MVP-T1。
  - [`plan.md`](./plan.md)：POC 待办全部勾选；下一步改为写 MVP-T1 AC。
  - [`agent-stories.md`](./agent-specs/agent-stories.md)：`agent-poc-01` 状态 Done；范围注明全环超出原 ADR-054 D2。
  - [`real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)：POC 签收行改为已完成。
  - 验证脚本补质量 mock（时钟链 / 下午站 / 餐店卡）与检查项；产物 [`poc-true-agent-verification.html`](./poc-true-agent-verification.html)。
  - 知识：[`knowledge/agent/poc-verify-fixture-quality.md`](./knowledge/agent/poc-verify-fixture-quality.md)。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`places-agent/scripts/verify-poc-true-agent.ts`。

## 2026-09-07 — POC 全环模型驱动 + 无 Google 验证

- **范围：** `plan_trip` 全环改为模型 act-or-stop；fixture 验证不调 Google。
- **变更：**
  - `runFullLoopAgent` 为默认全环：模型选 `resolve_origin_stay` / `search_places` / `make_itinerary` / `plan_next_stop` / `commit_artifacts` / `stop`；全环发现的 eligible POI 回填 registry。
  - 旧 `runFullLoop` 固定管线保留为 `PLAN_TRIP_LEGACY_FULL_LOOP=1` 回退。
  - 新增 `places-agent/scripts/verify-poc-true-agent.ts`（进程内、fixture、Prisma Lisbon 池）；结果写入 [poc-true-agent-verification.md](./poc-true-agent-verification.md)。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) G8 行标记已实现。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`plan-trip.ts`。

## 2026-09-07 — 跨供应商同地点去重交由 LLM（ADR-058）+ 真智能体改进路线

- **范围：** stops pool 跨供应商去重策略；POC 真智能体合规路线。
- **变更：**
  - 新增 [ADR-058](./adr/ADR-058-cross-provider-duplicate-llm-judges.md)：跨供应商同一地点（香港 Google+AMAP）不写库合并、不读池硬去重，去重由环内 LLM 取点时自行判断；prompt 硬约束 + 事实闸兜底。
  - 更新 [agent-design.md](./agent-specs/agent-design.md) §9.2 / §5.2：明确 LLM 取点需自己判断跨供应商同地点。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 追加「下一步改进」节：列出 G6/G7/G8/F89/G5 + POC 签收待决项、建议顺序、结合 ADR-057 三层测试的验证矩阵。
- **关联：** ADR-049、ADR-050、ADR-052、ADR-056、ADR-057、ADR-058；`destination-poi-registry.ts`；`plan-trip.ts`。

## 2026-09-07 — 成本节约测试策略入库 + change-log 启动

- **范围：** agent 测试策略文档化；修订记录机制启动。
- **变更：**
  - 新增 [ADR-057](./adr/ADR-057-cost-conscious-agent-test-strategy.md)：L0 fixture CI、L1 probe 文件缓存、L2 日配额门、stops pool 作 live feed（非城市百科）。
  - 更新 [agent-test-plan.md](./agent-specs/agent-test-plan.md)：绑定表、§1.2、§9 CI 门控、§11 清单。
  - 新建本文件，作为此后 specs 修订的单一入口；[`README.md`](./README.md) Layout 表增加 `change-log.md` 行。
- **关联：** ADR-021、ADR-042、ADR-056、ADR-057；`.github/workflows/places-agent-tests.yml`；`places-agent/scripts/seed-city-pois.ts`、`probe-budget.ts`、`search-cache.ts`。

## 2026-09-07 — Registry 回填语义 + 三城 stops pool 种子

- **范围：** AttractionPoi 写路径与测试 feed。
- **变更：**
  - 新增 [ADR-056](./adr/ADR-056-registry-backfill-semantics.md)：唯一性 `(provider, native_id)`；匹配无差跳过 / 有差更新+aliases / 无匹配新增；`cardSlim` 存 photos、不存 `must_see`；`plan_trip` commit 接线 `safeUpsertEligiblePois`。
  - 更新 [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 挂钩表与落库节。
  - 种子脚本建 Lisbon / Hong Kong / Taipei 池各 ≥100（有图优先）；删除旧「里斯本」「香港」碎片 destination。
- **关联：** ADR-049、ADR-051、ADR-056；`destination-poi-registry.ts`、`plan-trip.ts`。
