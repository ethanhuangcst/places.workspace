# Places 家族产品 Backlog

**Status:** active · as_of 2026-09-07  
**Branch:** real-agent-refactory  
**Target design:** [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)  
**ADRs:** [`adr/`](./adr/) · especially ADR-039 (as-built vs target), ADR-042 (no city encyclopedia), ADR-050 Proposed (2play zero product LLM)

**产品级概要需求的唯一真源；过程知识与 ADR 外链到本文件。**

Acceptance criteria (GWT) live only in:
- [`agent-specs/agent-stories.md`](./agent-specs/agent-stories.md)
- [`2play-specs/2play-stories.md`](./2play-specs/2play-stories.md)
- [`2eat-specs/2eat-stories.md`](./2eat-specs/2eat-stories.md)

## §0 当前下一步

**MVP-24 暂停**（质量问题不可验收）。剩余 2play 开放行见 §1 状态 `Paused`，待真智能体主干稳定后再排期。

**真智能体重构插入（ADR-054 / ADR-055）：**

1. **POC** — `agent-poc-01` **Done**（2026-09-07）。Lisbon 单城：`plan_trip` 模型驱动全环（intake + 芯片 + 骨架 + fill + artifacts）；fixture 验证不调 Google。验收物 [`poc-true-agent-verification.html`](./poc-true-agent-verification.html)。
2. **下一步：MVP-T1** — `agent-itinerary-93a`（intake + need_input + 芯片）+ `2play-plan-90a`（5 题问卷渲染 + 第 6 题芯片勾选回传）。
3. **MVP-T2** — 补池 + 骨架 + 按日 filled（无餐）+ 2play 行程详情逐日渲染。
4. **MVP-T3** — 餐档 + directions + 硬闸 + 2play 行程详情含餐与交通。
5. **MVP-T4** — 四卡（artifacts）+ 2play 出行贴士页。
6. **MVP-T5** — chat 改行程 + 2play in-page chat。
7. **扩展探针** — 杭州（大陆 AMAP-only / 废除 discover 扩源 / D9-D10）、香港（Google+AMAP）、起点卡（ADR-053）。

一次一条故事（`incremental-delivery`）。POC 已签收；开始写 MVP-T1 AC。

## §1 功能表

| 产品 MVP | 产品 | 模块 | 功能编号 | 功能名 | 功能描述 | 状态 |
| --- | --- | --- | --- | --- | --- | --- |
| MVP-1 | agent | search | `agent-search-01` | 餐厅搜索 | 通过调用方请求的供应商按位置和条件搜索餐厅 | Done |
| MVP-1 | agent | details | `agent-details-03` | 地点详情 | 使用供应商原生地点 id 获取已知地点的详情 | Done |
| MVP-1 | agent | nav | `agent-nav-04` | 导航助手 | 为地点返回不含密钥的导航深度链接和 URL | Done |
| MVP-1 | agent | geocode | `agent-geocode-05` | 地理编码 | 按需对地址进行地理编码和反向地理编码 | Done |
| MVP-1 | agent | vendors | `agent-vendors-06` | 地图供应商选择 | 调用方传递 providers[]；验证凭据；不静默换供应商 | Done |
| MVP-1 | agent | card | `agent-card-07` | 地点卡来源 | 每张地点卡列出 sources[]；可选合并重复项 | Done |
| MVP-1 | agent | http | `agent-http-11` | HTTP API 和 MCP | HTTP API 与 MCP 提供相同工具；标识为 places-agent | Done |
| MVP-1 | agent | auth | `agent-auth-12` | 调用方 API 密钥认证 | 仅向通过调用方 API 密钥认证的调用方提供服务 | Done |
| MVP-1 | agent | i18n | `agent-i18n-13` | 双语输出 | 智能体用户可见输出支持 EN/CN/HK/TW | Done |
| MVP-1 | agent | admin | `agent-admin-14` | 管理员首页 | 公开首页：指令链接与管理员登录控件 | Done |
| MVP-1 | agent | admin | `agent-admin-15` | 管理员登录和用户 | 管理员登录、默认管理员、Resend 重置与邀请 | Done |
| MVP-1 | agent | admin | `agent-admin-16` | 管理员落地页 | 登录后左侧导航；头部指令链接与问候 | Done |
| MVP-1 | agent | admin | `agent-admin-17` | 调用方 API 密钥 | 创建、编辑、重新生成、删除调用方 API 密钥 | Done |
| MVP-1 | agent | admin | `agent-admin-18` | 智能体指令 | 调用 places.agent-mate.ai 的方法说明页 | Done |
| MVP-1 | agent | admin | `agent-admin-19` | 管理应用 i18n | 管理 UI 和邮件支持四 locale | Done |
| MVP-1 | 2play | header | `2play-header-01` | App header & navigation | Sticky header：logo、行程规划 / 我的行程 / 个人信息、active、移动 Menu | Done |
| MVP-1 | 2play | header | `2play-header-02` | 已登录用户 chrome | 问候、avatar、登出 | Done |
| MVP-1 | 2play | header | `2play-header-03` | Locale switcher (app) | EN / CN / HK / TW | Done |
| MVP-1 | 2play | footer | `2play-footer-04` | Family footer (app) | places.family 行（App 底纹） | Done |
| MVP-1 | 2play | footer | `2play-footer-05` | Family footer (public) | places.family 行（公开页） | Done |
| MVP-1 | 2play | i18n | `2play-i18n-06` | Four-locale catalogs | 全部用户可见字符串为 key；四 locale（含 plan progressive preview_*） | Done |
| MVP-1 | 2play | home | `2play-home-07` | Public landing | Headline、lead、注册/登录 CTA | Done |
| MVP-1 | 2play | account | `2play-account-08` | Register | 创建账号：必填姓名/邮箱/密码；选填性别年龄出发地兴趣 | Done |
| MVP-1 | 2play | account | `2play-account-09` | Sign in | 邮箱密码登录；失败提示 | Done |
| MVP-1 | 2play | account | `2play-account-10` | Reset password | 请求重置邮件 | Done |
| MVP-1 | 2play | account | `2play-account-11` | Set password | 从链接设新密码；过期态 | Done |
| MVP-1 | 2play | profile | `2play-profile-12` | User profile | 单卡：资料 + 出行兴趣（多选）；独立保存 | Done |
| MVP-1 | 2play | profile | `2play-profile-13` | Required-field markers | 个人信息必填项标 * 与说明 | Done |
| MVP-1 | 2eat | header | `2eat-header-01` | App header & navigation | Sticky header on signed-in pages: logo, Decide / Saved / Profile, active section, mobile menu | Done |
| MVP-1 | 2eat | header | `2eat-header-02` | 已登录用户 chrome | Greeting with display name, avatar initial, Log out | Done |
| MVP-1 | 2eat | header | `2eat-header-03` | Locale switcher (app) | EN / CN / HK / TW switcher in the app header | Done |
| MVP-1 | 2eat | footer | `2eat-footer-04` | Family footer (app) | places.family row on signed-in pages, styled like the app header | Done |
| MVP-1 | 2eat | footer | `2eat-footer-05` | Family footer (public) | places.family row on public pages, seamless with the page cloth | Done |
| MVP-1 | 2eat | i18n | `2eat-i18n-06` | Four-locale catalogs | All user-visible strings via i18n keys; EN / CN / HK / TW | Done |
| MVP-1 | 2eat | home | `2eat-home-07` | Public landing | Question-led home, mini preview cards, register and sign-in CTAs | Done |
| MVP-1 | 2eat | account | `2eat-account-08` | Register | Create account with personal fields, default location, password, optional photo | Done |
| MVP-1 | 2eat | account | `2eat-account-09` | Sign in | Email and password sign-in; failed sign-in message | Done |
| MVP-1 | 2eat | account | `2eat-account-10` | Reset password | Request password reset email | Done |
| MVP-1 | 2eat | account | `2eat-account-11` | Set password | Set new password from invite or reset link; expired link state | Done |
| MVP-1 | 2eat | profile | `2eat-profile-12` | Personal information | Edit name, email, gender, age, default location, optional photo; separate save | Done |
| MVP-1 | 2eat | profile | `2eat-profile-13` | Tastes & constraints | Cuisine likes/dislikes, spice, party size, hard constraints, meal contexts; separate save | Done |
| MVP-2 | agent | search | `agent-search-02` | 地点搜索 | 以相同方式搜索非餐厅地点（景点、POI） | Done |
| MVP-2 | agent | enrich | `agent-enrich-08` | Tripadvisor 丰富化 | 按名称+位置匹配可选 Tripadvisor 评分/内容 | Done |
| MVP-2 | agent | itinerary | `agent-itinerary-09` | 行程规划 | 多站点结构化行程（LLM/legacy）；2play 主路径不调本工具 | Done |
| MVP-2 | agent | chat | `agent-chat-10` | 自然语言地点聊天 | OPENAI_CN 工具循环自然语言地点聊天 | Done |
| MVP-2 | 2eat | decide | `2eat-decide-14` | Search criteria | Area or pin, meal context, budget per person, optional craving; Find restaurants | Done |
| MVP-2 | 2eat | decide | `2eat-decide-15` | Results header & summary | Results title with last-updated time; showing X–Y of Z summary | Done |
| MVP-2 | 2eat | decide | `2eat-decide-16` | Reshuffle | Same filters; show next picks from the current short list | Done |
| MVP-2 | 2eat | decide | `2eat-decide-17` | Results pagination | Numbered pages with Previous / Next over the short list | Done |
| MVP-2 | 2eat | decide | `2eat-decide-18` | Pick cards | Photo, facts, source ids, fit badge, walk time / warnings; Details and Open map | Done |
| MVP-2 | 2eat | decide | `2eat-decide-19` | Empty & partial results | Empty state with widen-filters action; partial vendor banner | Done |
| MVP-2 | 2eat | place | `2eat-place-21` | Place details — facts | Dialog with image + restaurant facts and honest missing states | Done |
| MVP-2 | 2eat | place | `2eat-place-22` | Place details — why | Why it fits, also nearby alternatives, menu/allergen disclaimer | Done |
| MVP-2 | 2eat | place | `2eat-place-24` | Open map & save | Secret-free map link; Save / Unsave from details | Done |
| MVP-2 | 2eat | saved | `2eat-saved-25` | Saved places list | Saved pick cards; Details; empty state | Done |
| MVP-2 | 2eat | saved | `2eat-saved-26` | Unsave | Remove a place from saved list or details | Done |
| MVP-2 | 2play | plan | `2play-plan-14` | Planner form | 三列边界表单；生成一条行程（L1 discover + BFF OPENAI_CN L2） | Done |
| MVP-2 | 2play | plan | `2play-plan-15` | Planner validation | 目的地/天数/起始日期必填；天数范围；时间成对校验 | Done |
| MVP-2 | 2play | plan | `2play-plan-16` | Itinerary day/hour view | Day tabs、Highlights、时段行、交通段、配图与外链；生成中 liveSlots | Done |
| MVP-2 | 2play | plan | `2play-plan-17` | Single itinerary only | 每次规划/重新规划只交付一条；无多卡短名单 | Done |
| MVP-2 | 2play | plan | `2play-plan-18` | Prefill from interests | Profile 出行兴趣可预填 Plan 偏好 chips | Done |
| MVP-2 | 2play | plan | `2play-plan-19` | Combo full options | 自定义 combo 展开始终列出全部选项 | Done |
| MVP-2 | 2play | saved | `2play-saved-20` | Saved trips grid | 仅已保存多卡；空态 | Done |
| MVP-2 | 2play | saved | `2play-saved-21` | Open saved trip | 详情 Day/Hour；无未保存 History | Done |
| MVP-2 | 2play | saved | `2play-saved-22` | Unsave trip | 从详情取消收藏 | Done |
| MVP-4 | 2play | chat | `2play-chat-24` | Local draft transcript | 回合写入 localStorage；刷新保留；登出清除 | Done |
| MVP-2 | 2play | plan | `2play-plan-30` | Progressive generate UX | 四段 UI：日提示 / slot_preview 细节提示 / 逐条 slot / pending skeleton | Done |
| MVP-3 | 2play | plan | `2play-plan-31` | Mode H prompt source | BFF 从 agent execution=host 拉 prompt；OPENAI_CN 执行；UI 契约不变 | Done |
| MVP-3 | 2play | plan | `2play-plan-32` | Arrange OPENAI_CN stream | L2 stream: true + 增量 parse；首 slot_preview 早于整日 JSON | Done |
| MVP-3 | 2play | plan | `2play-plan-33` | Real transit in timeline | 消费真 navigate/directions（非估时合成 transit） | Done |
| MVP-3 | 2eat | decide | `2eat-decide-20` | List-level agent chat | Floating chat panel scoped to current list and filters; transcript in browser-local storage only | Done |
| MVP-3 | 2eat | place | `2eat-place-23` | Place-level agent chat | Chat inside details dialog about this restaurant; transcript per place in browser-local storage only | Done |
| MVP-3 | 2eat | history | `2eat-history-27` | Decision history | Recent places you went to, meal context, re-run Decide; empty state | Done |
| MVP-4 | 2eat | decide | `2eat-decide-28` | Results sort | Re-order the short list by rank, rating, distance, or price level | Done |
| MVP-4 | 2eat | chat | `2eat-chat-29` | Resizable chat panel | List chat panel: sticky composer, no dead space; drag resize with current size as minimum | Done |
| MVP-4 | 2eat | chat | `2eat-chat-30` | Rich agent replies | Structured assistant messages with pick cards, photos, and external map links (new tab) | Done |
| MVP-4 | 2eat | chat | `2eat-chat-31` | Pending + place chat scroll | Waiting indicator while agent replies; place chat transcript scrolls inside a fixed chat box | Done |
| MVP-4 | 2eat | decide | `2eat-decide-32` | Show price on cards | Display agent price_level (and optional price_per_person) on pick cards and place details | Done |
| MVP-4 | 2eat | decide | `2eat-decide-33` | Keep location draft | Area/pin input keeps the latest typed value across locale switches | Done |
| MVP-4 | 2eat | decide | `2eat-decide-34` | Keep other criteria drafts | Meal context, budget, craving keep latest input across locale switches | Done |
| MVP-4 | 2eat | chat | `2eat-chat-35` | Persist chat panel size | Remember list chat width/height in localStorage across refresh | Done |
| MVP-3a | agent | vendors | `agent-vendors-20` | 供应商自动选择 | 按目的地+语言自动选择 provider 组合；caller 可覆盖 | Done |
| MVP-3a | agent | infra | `agent-infra-21` | 服务器稳定性 | JSON 解析安全、graceful shutdown、session TTL 清理 | Done |
| MVP-3b | agent | photo | `agent-photo-24` | 照片与价格档 | 搜索卡片返回 photos 与归一化 price_level | Done |
| MVP-3c | agent | geocode | `agent-geocode-25` | Geocode-first 与 Directions fallback | Provider 判定以 Geocode 为准；Directions Worker fallback | Done |
| MVP-4a | agent | i18n | `agent-i18n-26` | 语言路由与搜索关键词 | 按 locale/CJK 路由语言；搜索关键词多语言映射 | Done |
| MVP-4b | agent | infra | `agent-infra-27` | 搜索缓存与并行 | Geocode/search 短期缓存；行程餐食/多天搜索并行 | Done |
| MVP-5 | agent | admin | `agent-admin-28` | Admin API 加固 | Prisma 错误映射、Error Boundary、reset 4h、session iat | Done |
| MVP-6 | agent | itinerary | `agent-itinerary-29` | Prompt 组装器 | base.{en,zh} + overlays 拼接系统 prompt | Done |
| MVP-6 | agent | itinerary | `agent-itinerary-30` | LLM 行程规划 | 单日/多日 LLM+Zod；失败可降级 | Done |
| MVP-6 | agent | itinerary | `agent-itinerary-31` | 行程 MCP 拆分 | MCP+HTTP discover_places / arrange_day | Done |
| MVP-7 | agent | itinerary | `agent-itinerary-32` | 行程 MCP P0 止损 | date nullish；arrange 入模 slim；MCP description 互斥 | Done |
| MVP-7 | agent | discover | `agent-discover-33` | Discover 候选质量 | L1 无 LLM：过滤、cluster 去重、池头多样性（ADR-038） | Done |
| MVP-8 | agent | discover | `agent-discover-34` | Discover 候选质量（Arm A） | 通用模板填池 + Google RELEVANCE；must-see LLM 推断（ADR-042） | Done |
| MVP-8 | agent | arrange | `agent-arrange-35` | Arrange Mode H handoff | execution=host 返回 prompt，本请求不调 LLM | Done |
| MVP-8 | agent | arrange | `agent-arrange-36` | L2 硬必去 | 必去类必须出现；漏排硬失败重试 | Done |
| MVP-8 | agent | nav | `agent-nav-37` | 行程真交通 | directions 结果写入行程时间线 | Done |
| MVP-8 | agent | infra | `agent-infra-38` | MCP SSE session | 修复 POST /sse Streamable session 可恢复 | Done |
| MVP-9 | agent | infra | `agent-infra-40` | MCP arrange 服务端硬闸 | Cancelled：MVP-10 删 arrange_day | Cancelled |
| MVP-9 | agent | arrange | `agent-arrange-42` | Arrange 输出校验三件套 | 站间时序 / 同日餐厅去重 / day-trip 补搜；迁入 F44 | Done |
| MVP-10 | agent | itinerary | `agent-itinerary-43` | make_itinerary 轻骨架 | 一次 LLM 多日 stop-order 骨架；NDJSON skeleton_* | Done |
| MVP-10 | agent | fill | `agent-fill-44` | plan_next_stop / display 填充 | 无 LLM 逐站 transit + 卡片；F42 校验迁入 | Done |
| MVP-10 | agent | infra | `agent-infra-47` | MCP 骨架 host_instructions | discover→make→fill 链指令 | Done |
| MVP-11 | agent | visa | `agent-visa-48` | 签证要求查询 | Orizn REST adapter；MCP/HTTP visa_requirement（ADR-044） | Done |
| MVP-12 | agent | iconic | `agent-iconic-49` | findIconicPlaces 双模 | grounded/ungrounded 必去（ADR-045） | Done |
| MVP-12 | agent | tips | `agent-tips-50` | travel_tips | 目的地 tips 工具（ADR-045） | Done |
| MVP-12 | agent | infra | `agent-infra-51` | 别名重指向 make_itinerary | plan_itinerary/trip_plan/trips → 骨架流 | Done |
| MVP-12 | agent | infra | `agent-infra-52` | MCP /mcp stateless | 无会话化（ADR-045） | Done |
| MVP-13 | agent | fill | `agent-fill-53` | 填充时钟 end_time | next_tool_call 传递 end_time；时段累加 | Done |
| MVP-13 | agent | meal | `agent-meal-54` | 餐位窗口 | display 按 meal_slot 锚定午餐/晚餐窗 | Done |
| MVP-13 | agent | itinerary | `agent-itinerary-55` | 骨架超节奏裁剪 | 超节奏日确定性裁 attraction 后重校验 | Done |
| MVP-13 | agent | itinerary | `agent-itinerary-56` | 站名归一化 | 站名归一化匹配候选池规范名 | Done |
| MVP-13 | agent | discover | `agent-discover-57` | 区域 must_include 展开 | 区域型 must_include geocode + nearby 子搜索 | Done |
| MVP-13 | agent | itinerary | `agent-itinerary-58` | make_itinerary 失败 detail | 失败 envelope data.detail + host 可读回退 | Done |
| MVP-14 | agent | fill | `agent-fill-59` | stay 角色 | 区分 day_origin vs return stay；非 origin stay 累加时钟 | Done |
| MVP-14 | agent | fill | `agent-fill-60` | 交通地理/时长闸 | geocode 带 city + 距离/时长闸；拒区域名单站 | Done |
| MVP-14 | agent | meal | `agent-meal-61` | 迟到午餐重座 | 迟到 lunch 升 dinner；骨架 lunch 不在末站 | Done |
| MVP-15 | agent | itinerary | `agent-itinerary-62` | 骨架确定性修复 | reseatStay + dropCity + 超时 prior validation | Done |
| MVP-16 | agent | trip | `agent-trip-63` | Trip Store | PG+内存；懒创建；revision（ADR-046） | Done |
| MVP-16 | agent | trip | `agent-trip-64` | fetch_trip_details | 按 fields 只读切片 | Done |
| MVP-16 | agent | trip | `agent-trip-65` | 删除 display_current_stop | 写并入 plan_next_stop；读走 fetch | Done |
| MVP-3r | 2play | plan | `2play-plan-34` | Boundary passthrough | BFF body 组装透传全部 PlanBoundaries 到 discover + arrange | Done |
| MVP-3r → MVP-10 | 2play | plan | `2play-plan-35` | Origin geocode before enrich | enrich 前 geocode — Superseded → MVP-10 plan-46 / agent F44 | Superseded |
| MVP-3r | 2play | plan | `2play-plan-36` | Keep LLM transit fields | daySchema/blockSchema 保留 transit 字段；enrich 失败显式降级；2play 侧 F42 等价 | Done |
| MVP-18 | agent | itinerary | `agent-itinerary-71` | plan-takeoff 预算默认 | 预算默认 mid；顶栏 i18n「适中」 | Done |
| MVP-18 | agent | itinerary | `agent-itinerary-72` | 骨架预览 UX | 预览用 fetch skeleton：折叠每日相同酒店 stay | Done |
| MVP-18 | agent | iconic | `agent-iconic-74` | findIconicPlaces 必去质量 | 热度排序 + 多天空间多样性；无城市表 | Done |
| MVP-18 | agent | trip | `agent-trip-75` | trip-host-fetch | 每步写完 2play fetch_trip_details；展示来自 store | Done |
| MVP-18 | agent | trip | `agent-trip-76` | artifacts-tips-visa | travel_tips / visa_requirement 双写 artifacts | Done |
| MVP-18 | agent | itinerary | `agent-itinerary-77` | intake 时刻容错 | 口语时刻 ABC 三层解析；失败默认 09:00 | Done |
| MVP-19 | agent | itinerary | `agent-itinerary-78` | make 墙钟可恢复 | make_itinerary 超时对齐网关；失败先 fetch skeleton | Done |
| MVP-19 | agent | discover | `agent-discover-79` | 池内热度打标 | discover 先池后热度打标；slim 保留评论数 | Done |
| MVP-19 | agent | itinerary | `agent-itinerary-80` | 骨架禁 stay-only 日 | 必去只对 stop.name；池有 attraction 时每天至少一站非 stay | Done |
| MVP-19 | agent | trip | `agent-trip-81` | trip 无 watch 同流 fetch | 写失败→fetch；写成功→fetch；session 持久化 trip_id | Done |
| MVP-19 | agent | discover | `agent-discover-82` | must_see 正交 | must_see=热门；constraints.must_include=用户 | Done |
| MVP-21 | agent | itinerary | `agent-itinerary-83` | 骨架地理锚点=目的地 | 过滤锚点=城市 geocode；origin 过远丢 lat/lng（ADR-048） | Done |
| MVP-22 | agent | discover | `agent-discover-84` | 可规划景点门槛 | L0 谓词过滤合称/名胜区；must_include 降级不 502（ADR-049） | Done |
| MVP-22 | agent | meal | `agent-meal-85` | 骨架餐档无店名 | make 只排景点+餐档 slot；无餐馆店名 | Done |
| MVP-22 / MVP-23 | agent | meal | `agent-meal-86` | 填站邻站搜餐 | plan_next_stop 邻域搜餐或 meal_skipped；23 加严顺路现搜 | Done |
| MVP-23 / ADR-053 | agent | fill | `agent-fill-88` | 起点 stay 整卡 | 选定酒店身份钉死；fill 有指针不重搜；photos[0] maxWidthPx=800（ADR-053/051） | Done |
| MVP-23 / ADR-052 | agent | discover | `agent-discover-89` | 大陆 discover 禁扩源 | 大陆 discover 仅 AMAP；详情同身份（ADR-052） | Done |
| MVP-23 | agent | fill | `agent-fill-90` | 审天（不因超时砍站） | 审天：重复店、绕路对调未填；不因超时砍景点（F90-1） | Done |
| MVP-23 | agent | meal | `agent-meal-91` | 指针/餐窗/停留 | 骨架 provider+native_id；餐窗新定义；禁止 meal_skipped；孤立停留 45/60 | Done |
| MVP-23 | agent | fill | `agent-fill-92` | 起点 + 按景点坐标搜餐 | stay 用 intake 起点或城市；搜餐 near=当天景点坐标非酒店 | Done |
| MVP-19 | 2play | plan | `2play-plan-40` | 助手叙事 + 同流 fetch | 每步叙事：搜点/骨架/填站；骨架先出再 fill；session trip_id（MVP-19） | Done |
| MVP-9 · 停放 | agent | infra | `agent-infra-39` | Typecheck 清零 | 预存 tsc 错误清零，恢复 make quality typecheck 门 | ToDo |
| MVP-9 · 停放 | agent | infra | `agent-infra-41` | Opt-in 分层 + runbook | E2E-live 边界裁剪；caller E2E runbook；build warning 清单 | ToDo |
| MVP-10 · 24-P2c（硬删 gate 37 usable） | agent | itinerary | `agent-itinerary-45` | 工具清理 | navigate 已删；arrange_day/enrich 硬删 gate plan-46 | Done(producer)/ToDo(consumer) |
| MVP-16 · 24-P2c（硬删对齐） | agent | trip | `agent-trip-66` | 对外工具精简 | 评估并落地删/合并（硬删仍 gate plan-46） | Done(producer)/ToDo(consumer) |
| MVP-17 · 24-P0（契约续） | agent | fill | `agent-fill-67` | plan_next_stop fill 契约 | end_time HH:MM；omit null revision；invalid_input i18n；revision 冲突重试 | Paused |
| MVP-18 P2 · 24-P2a | agent | itinerary | `agent-itinerary-68` | plan-nav chips CSS | .plan-nav__quick wrap + chip nowrap；mock.css 同步 | Paused |
| MVP-17 · 24-P0（展示源） | agent | iconic | `agent-iconic-69` | 必去地单一源 | 贴士 01 与助手步骤 g 同源 artifacts.tips.iconic_places；禁 merge discover | Paused |
| MVP-18 P2 · 24-P2a | agent | tips | `agent-tips-70` | travel-tips 文案 UI | 四卡排版 i18n；intro/必去地来自已 fetch artifacts | Paused |
| MVP-18 P2 · 24-P2a | agent | itinerary | `agent-itinerary-73` | plan-46 测对齐 | api-plan 对齐 skeleton NDJSON；fill/iconic/chip 测 | Paused |
| MVP-22 · 24-P1c（usable 探针） | agent | discover | `agent-discover-87` | 目的地景点库 | PG Destination+AttractionPoi 运行时库；不扩 CATALOG | In progress |
| MVP-24 · 24-P0b | agent | itinerary | `agent-itinerary-94` | AC29 失败文案诚实化 | make 非超时失败勿一律「框架超时」；timeout vs make_failed vs provider | Done |
| MVP-24 · 24-P0c | agent | fill | `agent-fill-95` | 打卡串停留时长 leftover | 连续景点停留时长仍偏短（F88 续 / 23-S2） | Done |
| MVP-24 · 24-P0d | agent | infra | `agent-infra-96` | 主 LLM 路径 ops | 百炼开通 qwen-plus 或默认 OPENAI_CN；少烧 403 | Done |
| MVP-24 · 24-P0 / Target | agent | discover | `agent-discover-97` | Feature 89 消费端核对 | 大陆 AMAP-only discover / 详情同身份在 2play 消费路径核对（若仍需） | Done(producer)/ToDo(consumer) |
| MVP-4 · 24-P4a | 2play | chat | `2play-chat-23` | In-page plan chat | Plan 下方唯一 Chat；BFF 本应用 OPENAI_CN 流式改当前行程（ADR-036） | Paused |
| MVP-2 AC1 Done / MVP-4 AC2–3 · 24-P4b | 2play | plan | `2play-plan-25` | Save itinerary + chat | AC1：保存行程（messages 可 []）；AC2–3：保存含对话快照 | Done(producer)/ToDo(consumer) |
| MVP-4 · 24-P4c | 2play | saved | `2play-saved-26` | DB chat snapshot | 打开已保存行程可读 DB 对话；只读提示 | Paused |
| MVP-5 · 24-P5a | 2play | plan | `2play-plan-27` | Replan with confirm | 确认后换新行程（同新管线 make/fill）；保留 local chat + 分隔提示 | Paused |
| MVP-5 · 24-P5b | 2play | plan | `2play-plan-28` | Export PDF | 基于当前行程事实导出；不编造场所 | Paused |
| MVP-5 · 24-P5c | 2play | chat | `2play-chat-29` | Chat height resize | SE 把手仅调整高度；尊重最小高度 | Paused |
| MVP-10 · 24-P0a / P0-ui / P1 / P2 | 2play | plan | `2play-plan-37` | MVP-10 轻骨架消费端 | 5 字段 + Travor UI + 助手 + 新 BFF 管线（make_itinerary → 逐 stop 填充） | Paused |
| MVP-11 · 24-P3a | 2play | profile | `2play-profile-38` | Nationality field | 注册/资料页国籍下拉（ISO alpha-3，选填）；持久化至 User；四 locale i18n | Paused |
| MVP-11 · 24-P3b | 2play | plan | `2play-plan-39` | Travel advice visa slot | 出行建议页预留签证信息展示位；消费 agent visa_requirement（本切片仅 spec/mock 占位） | Paused |
| MVP-20 · 24-P0a（usable 并入 37f） | 2play | plan | `2play-plan-41` | 行程规划页重建 | CTA→助手接管；静默 discover∥intake；make+fetch 骨架；起点目的地内确认 | Paused |
| MVP-24 · 24-P3c | 2play | plan | `2play-plan-94` | 签证运行时展示 | BFF → visa_requirement → artifacts → fetch 展示（非仅占位） | Paused |
| POC · true-agent | agent | poc | `agent-poc-01` | 真智能体探针（Lisbon 单城） | plan_trip 模型驱动全环（intake + 芯片 + 骨架 + fill + artifacts）；fixture 脚本/HTML 可观测；无 UI；检查表 #1/#5/#8/#25/#28 | Done |
| MVP-T1 · true-agent | agent | itinerary | `agent-itinerary-93a` | plan_trip intake + need_input + 芯片 | 对外 plan_trip：城市→懒建 trip_id→必去芯片→need_input；fetch candidates | ToDo |
| MVP-T1 · true-agent | 2play | plan | `2play-plan-90a` | 5 题问卷 + 第 6 题芯片 | 渲染 agent need_input；第 6 题 fetch 验真 must_see 勾选回传；无产品 LLM | ToDo |
| MVP-T2 · true-agent | agent | itinerary | `agent-itinerary-93b` | 补池 + 骨架 + 按日 filled（无餐） | 环内 search_places 补池；骨架分日；按日 filled；硬闸 | ToDo |
| MVP-T2 · true-agent | 2play | plan | `2play-plan-90b` | 行程详情逐日渲染 | fetch skeleton/filled 逐日展示；route-spine | ToDo |
| MVP-T3 · true-agent | agent | itinerary | `agent-itinerary-93c` | 餐档 + directions + 硬闸 | 餐档现搜；directions 腿；硬闸复查；ready/failed | ToDo |
| MVP-T3 · true-agent | 2play | plan | `2play-plan-90c` | 行程详情含餐与交通 | fetch filled 含餐店与交通推荐展示 | ToDo |
| MVP-T4 · true-agent | agent | tips | `agent-tips-93d` | 四卡（artifacts） | tips/visa 内部 adapter；一次 tips-prose；写入 artifacts | ToDo |
| MVP-T4 · true-agent | 2play | plan | `2play-plan-90d` | 出行贴士页 | fetch artifacts 四卡展示 | ToDo |
| MVP-T5 · true-agent | agent | chat | `agent-chat-93e` | chat 改行程 | plan_trip 同环；trip_id+自然语言→commit_trip 补丁/重排 | ToDo |
| MVP-T5 · true-agent | 2play | chat | `2play-plan-90e` | in-page chat | /api/chat 转发 agent plan_trip；无本地模型补全 | ToDo |
| 扩展 · true-agent | agent | discover | `agent-discover-93f` | 杭州/香港/起点卡探针 | 大陆 AMAP-only + 废除扩源 + D9/D10；HK 双源；ADR-053 起点整卡 | ToDo |
| 扩展 · true-agent | 2play | plan | `2play-plan-90f` | 三城 + 起点卡消费 | 三城行程 + 起点卡展示与 fill 抄卡 | ToDo |

### Legend

| 值 | 含义 |
| --- | --- |
| Done | Shipped and accepted for this family row |
| ToDo | Not started or not yet Done |
| In progress | Active work |
| Superseded | Replaced by a later story / MVP; keep for traceability |
| Cancelled | Explicitly dropped |
| Done(producer)/ToDo(consumer) | Agent (or producer) side Done; consumer product still open (ADR-039) |
| Paused | On hold pending true-agent trunk stabilization (ADR-054/055); not scheduled |

## §3 共同规划原则

- One user story to DoD (`incremental-delivery`)
- Producer Done ≠ consumer Done (ADR-039)
- what2eat isolation: not folded into `plan_trip` (ADR-050 D3)
- No city encyclopedia in source (ADR-042)
- True-agent Target = `plan_trip` + `fetch_trip_details` (ADR-050 Accepted); MVP-24 as-built polish Paused pending true-agent trunk (ADR-054/055)
- MVP slicing = true-agent capability + 2play consumption closed loop per batch (ADR-055); POC before UI (ADR-054)
- Family ID = `{product}-{module}-{NN}`; stories keep product-local codes as anchors
- Detail AC only in the three stories files; this file owns schedule, status, and product-level overview

## §4 文档职责

| Doc | Role |
| --- | --- |
| [`product-backlog.md`](./product-backlog.md) | **唯一**产品级概要需求与排期真源（本文件） |
| `*-stories.md` | GWT / AC only |
| [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md) | Target architecture design |
| [`adr/`](./adr/) | Binding decisions |
| [`knowledge/`](./knowledge/) | Process lessons; historical refactor log: [`knowledge/agent/refactor-plan-archive.md`](./knowledge/agent/refactor-plan-archive.md) |

## §5 产品定义

### places-agent

places-agent is a multi-provider place gateway and agent on one host: search, details, geocode, navigation, discovery, itinerary tools, natural-language place chat, visa helpers, and an **operator admin web app**. It does **not** own consumer UX (what2eat / where2play).

Public host: `places.agent-mate.ai` (operator HTML + HTTP API + MCP). Caller-visible agent id: `places-agent`. Callers include what2eat.food, where2play.place, and third parties via MCP.

**Capabilities (tool names):** `search_restaurants`, `search_places`, `get_place_details`, `geocode`, `navigate` / directions, `discover_places`, `make_itinerary`, `plan_next_stop`, `fetch_trip_details`, `chat`, `visa_requirement`. Target adds `plan_trip` ([ADR-050](./adr/ADR-050-where2play-no-product-llm.md)).

**Provider selection:** For planning paths, omit caller `providers[]`; the agent auto-routes map vendors per [ADR-052](./adr/ADR-052-map-provider-routing.md).

**Transports:**
- **HTTP** — first-party BFFs with caller API key.
- **MCP** — same tool semantics for agent hosts; same caller API key.
- **Operator web** — same-origin admin APIs with admin session (not caller key).

**Non-goals:** what2eat / where2play screens and product UX; deploy topology → [`2.architecture.md`](./2.architecture.md); city encyclopedia / per-city must-see tables in source ([ADR-042](./adr/ADR-042-no-city-encyclopedia-in-source.md)).

**AC:** [`agent-specs/agent-stories.md`](./agent-specs/agent-stories.md) · **Target design:** [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)

### what2eat

**what2eat** (`what2eat.food`) shortlists restaurants from saved preferences plus **Decide** (location, meal context, budget, optional craving). Place facts come from places-agent via the BFF; the browser never holds map, caller, or LLM keys.

| Layer | Owns | Does not own |
| --- | --- | --- |
| **Web app** | Account, prefs, Decide UX, cards/details, Saved/History, match copy, chat UX, **browser-local chat transcript** | Map adapters, agent admin UI |
| **BFF (same origin)** | Session, map deeplink choice, preference matching, chat orchestration (stateless) | Vendor secrets; **persisted chat** |
| **places-agent** | `search_restaurants`, `get_place_details`, geocode, `sources[]`, optional Tripadvisor enrich, `chat` | Consumer UI, preference profile |

```text
Browser → what2eat /api/* → places-agent HTTP (/v1)
Browser → what2eat /api/chat → places-agent /v1/chat (not MCP)
```

BFF uses **HTTP only** (not MCP). Primary search path: `search_restaurants`; chat explains, does not replace search. List/detail chat transcripts live in **browser localStorage** only.

**i18n:** All user-visible strings are **i18n keys**. Locales: `EN`, `CN` (zh-CN), `HK` (zh-HK), `TW` (zh-TW); HK and TW catalogs are separate.

**Isolation:** what2eat is isolated from true-agent `plan_trip` ([ADR-050](./adr/ADR-050-where2play-no-product-llm.md) D3). It keeps `geocode` / `search_restaurants` / `get_place_details` / `chat`.

**Non-goals:** Order / pay, menu or allergen authority, medical diet advice; `plan_itinerary` / `plan_trip`; places-agent admin UI; browser-held map / caller / LLM keys; browser MCP; server- or DB-stored chat; cross-device chat sync.

**AC:** [`2eat-specs/2eat-stories.md`](./2eat-specs/2eat-stories.md) · **Design:** [`2eat-specs/2eat-design.md`](./2eat-specs/2eat-design.md)

### where2play

**where2play** (`where2play.place`) helps users who do not know where to go: light travel interests in **Profile**, trip bounds on **Plan**, one executable itinerary (day / hour), in-page chat, save to **Saved**, optional PDF export.

Treat **as-built** and **Target** as two states. Do not read Qwen L2 as the permanent design.

**As-built (until true-agent migration):** BFF may hold product **Qwen / OPENAI_CN** for L2 arrange and the trip assistant ([ADR-036](./adr/ADR-036-where2play-assistant-quanzil.md) / [ADR-037](./adr/ADR-037-where2play-plan-l2-quanzil.md) / [ADR-047](./adr/ADR-047-qwen-primary-llm.md)).

Pipeline: `discover_places` → `make_itinerary` → `plan_next_stop` × N → `fetch_trip_details` (after writes). Places and maps: BFF → places-agent HTTP + caller API key. One itinerary per plan / replan. Chat draft truth = localStorage; saved snapshot truth = App DB on Save. Browser never holds map / caller / LLM keys.

```text
Browser → /api/plan* → BFF: discover → make_itinerary → plan_next_stop×N → fetch_trip_details
Browser → /api/chat  → BFF product LLM (stream); optional agent search
```

**Target (ADR-050 Proposed):** where2play has **zero product LLM** ([ADR-050](./adr/ADR-050-where2play-no-product-llm.md)). Only `plan_trip` + `fetch_trip_details`. BFF renders `need_input`, posts answers, fetches trip slices. No local arrange / tips prose / trip-edit LLM on the BFF. Gate: `agent-itinerary-93` + `2play-plan-90`.

```text
Browser → /api/plan* → BFF: plan_trip (± need_input loop) → fetch_trip_details
No BFF trip-edit / tips / arrange LLM
```

| Layer | As-built owns | Target owns | Does not own (either) |
| --- | --- | --- | --- |
| **Web app** | Account, Profile, Plan page, Saved cards, in-page Chat UX, localStorage draft transcript, travel UI | Same UX; surfaces `need_input` and trip slices from agent | Map adapters, agent admin UI |
| **BFF (same origin)** | Session, deeplinks, **L2 arrange → product LLM**, **assistant → product LLM stream**, agent discover / fill calls, Save to DB, PDF | Session, deeplinks, **`plan_trip` / `fetch_trip_details` only**, render/post `need_input`, Save, PDF — **no product LLM** | Map vendor secrets; force-write chat every turn |
| **places-agent** | L1 `discover_places`, search/details/geocode/navigate/`sources[]`, `make_itinerary`, `plan_next_stop`, `fetch_trip_details` | `plan_trip` (+ internal intake/compose), `fetch_trip_details` | Consumer UI, user Profile |

BFF uses **HTTP only** to places-agent (not MCP).

**i18n:** All user-visible strings are **i18n keys**. Locales: `EN`, `CN`, `HK`, `TW`.

**Non-goals:** Order / pay, ticket or hotel inventory authority; what2eat SSO; dual chat / FAB global chat; multi-itinerary shortlist; browser-held map / caller / LLM keys; browser MCP; auto-persist every chat turn; places-agent admin UI; multiplayer realtime; offline map packs.

**Success criteria (short):** Register → Profile interests → Plan multi-day bounds → one Day/Hour itinerary → in-page chat (local draft) → replan with confirm → Save (trip + chat snapshot to DB) → open Saved card → optional PDF. Product is not a booking authority.

**AC:** [`2play-specs/2play-stories.md`](./2play-specs/2play-stories.md) · **Design:** [`2play-specs/2play-design.md`](./2play-specs/2play-design.md) · **Target:** [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)
