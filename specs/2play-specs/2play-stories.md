# where2play — 用户故事

**where2play**（`where2play.place`）产品 backlog 与验收标准（AC）。


| Related               | Location                                                                                                               |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| 产品规格                  | [`product-backlog.md`](../product-backlog.md) §5                                                                         |
| 设计规范                  | [2play-design.md](./2play-design.md)                                                                                 |
| 行程生成 / Progressive UX | [itinerary-design.md](./itinerary-design.md)                                                                         |
| 页面契约（设计 §3）           | [2play-design.md](./2play-design.md) §3                                                                              |
| 性能 / L1–L2 交叉         | [../agent-specs/performance.md](../agent-specs/performance.md) §11               |
| 测试计划                  | [2play-test-plan.md](./2play-test-plan.md)                                                                           |
| UI mock-up            | [ui-mockup/](./ui-mockup/)                                                                                           |
| 家族架构                  | [../2.architecture.md](../2.architecture.md)                                   |
| 行程引擎归属                | [../adr/ADR-008-itinerary-ownership.md](../adr/ADR-008-itinerary-ownership.md) |
| places-agent          | [../agent-specs/](../agent-specs/)                                               |
| 签证（Orizn）           | [ADR-044](../adr/ADR-044-orizn-visa-rest-adapter.md) · agent Feature **48** · 2play Feature **38–39** |


**家族排期与状态真源：** [`../product-backlog.md`](../product-backlog.md)（§1 发布表 · §0 当前下一步）。  
本文件只保留角色、术语与各功能的 GWT / AC。产品原编号（如 `plan-46`、`F44`、`header-01`）不变，作锚点。

## 人物角色


| 角色    | 谁                    | 价值                     |
| ----- | -------------------- | ---------------------- |
| 休闲出行者 | 已登录用户                | 快速得到**一条**可执行多日行程并在页内改 |
| 新访客   | 公开首页访客               | 了解产品并创建账号              |
| 回访用户  | 有 Profile / 已保存行程的用户 | 用兴趣预填偏好；打开旧行程与对话快照     |




## 术语


| 术语          | 含义                                         | 不是              |
| ----------- | ------------------------------------------ | --------------- |
| **场所事实**    | 名称、地址、时段、配图等，经 places-agent 来自 map vendors | Agent 建议文案或用户偏好 |
| **行程边界**    | Plan 表单字段（目的地、天数、节奏、偏好…）                   | Profile 里的轻量兴趣  |
| **出行兴趣**    | Profile/注册上的多选偏好；可带到规划器预填                  | 当次行程全部边界        |
| **当前行程**    | Plan 页中部展示的唯一一条 Day/Hour 行程                | 多卡短名单           |
| **Chat 草稿** | Plan 会话 transcript；真源 = localStorage       | 每轮自动写库          |
| **已保存快照**   | 用户点「保存」时写入 App DB 的行程 + 当时对话               | 未保存跨设备同步        |
| **重新规划**    | 确认后丢弃未保存行程、生成新一条；保留本机对话并插入分隔               | 删除已保存行程记录       |
| **行程日提示**   | `.plan-phase.is-busy`：整趟「正在安排第 d/N 天…」     | 候选池统计           |
| **行程细节提示**  | `.plan-slot-preview`：当前正在生成的一条（景点/交通/餐）    | 候选池 P/U 作主文案    |
| **行程**      | `.slot` 行：已落地时段                            | 整日同 tick 一次性刷屏  |
| **加载中提示**   | `.slot--pending` 同构 skeleton               | 虚线框占位           |
| **Mode H**   | agent `arrange_day` `execution=host` 返 prompt；2play OPENAI_CN 执行 LLM | agent execution=agent 内跑 LLM |




# 第一部分 — 产品 backlog

**列说明：** **编号** = backlog 序号（1–37，稳定 id）；表内按 **MVP-1→MVP-10** 批次排列，同批内按编号。


| 编号  | 模块      | Feature code | 功能名                      | 功能描述                                                                 | Story                                                 | MVP                              | 状态                      | itinerary 优化相关                  |
| --- | ------- | ------------ | ------------------------ | -------------------------------------------------------------------- | ----------------------------------------------------- | -------------------------------- | ----------------------- | ------------------------------- |
| 1 | Header | `header-01` | App header & navigation | Sticky header：logo、行程规划 / 我的行程 / 个人信息、active、移动 Menu | [§1](#1-header-header-01--app-header--navigation) | **MVP-1** | **Done** | — |
| 2 | Header | `header-02` | 已登录用户 chrome | 问候、avatar、登出 | [§2](#2-header-header-02--signed-in-user-chrome) | **MVP-1** | **Done** | — |
| 3 | Header | `header-03` | Locale switcher (app) | EN / CN / HK / TW | [§3](#3-header-header-03--locale-switcher-app) | **MVP-1** | **Done** | — |
| 4 | Footer | `footer-01` | Family footer (app) | places.family 行（App 底纹） | [§4](#4-footer-footer-01--family-footer-app) | **MVP-1** | **Done** | — |
| 5 | Footer | `footer-02` | Family footer (public) | places.family 行（公开页） | [§5](#5-footer-footer-02--family-footer-public) | **MVP-1** | **Done** | — |
| 6 | i18n | `i18n-01` | Four-locale catalogs | 全部用户可见字符串为 key；四 locale（含 plan progressive preview_*） | [§6](#6-i18n-i18n-01--four-locale-catalogs) | **MVP-1** | **Done** | P0（preview keys） |
| 7 | Home | `home-01` | Public landing | Headline、lead、注册/登录 CTA | [§7](#7-home-home-01--public-landing) | **MVP-1** | **Done** | — |
| 8 | Account | `account-01` | Register | 创建账号：必填姓名/邮箱/密码；选填性别年龄出发地兴趣 | [§8](#8-account-account-01--register) | **MVP-1** | **Done** | — |
| 9 | Account | `account-02` | Sign in | 邮箱密码登录；失败提示 | [§9](#9-account-account-02--sign-in) | **MVP-1** | **Done** | — |
| 10 | Account | `account-03` | Reset password | 请求重置邮件 | [§10](#10-account-account-03--reset-password) | **MVP-1** | **Done** | — |
| 11 | Account | `account-04` | Set password | 从链接设新密码；过期态 | [§11](#11-account-account-04--set-password) | **MVP-1** | **Done** | — |
| 12 | Profile | `profile-01` | User profile | 单卡：资料 + 出行兴趣（多选）；独立保存 | [§12](#12-profile-profile-01--user-profile) | **MVP-1** | **Done** | — |
| 13 | Profile | `profile-02` | Required-field markers | 个人信息必填项标 `*` 与说明 | [§13](#13-profile-profile-02--required-field-markers) | **MVP-1** | **Done** | — |
| 14 | Plan | `plan-01` | Planner form | 三列边界表单；生成一条行程（L1 discover + BFF OPENAI_CN L2） | [§14](#14-plan-plan-01--planner-form) | **MVP-2** | **Done** | P0·核心 |
| 15 | Plan | `plan-02` | Planner validation | 目的地/天数/起始日期必填；天数范围；时间成对校验 | [§15](#15-plan-plan-02--planner-validation) | **MVP-2** | **Done** | — |
| 16 | Plan | `plan-03` | Itinerary day/hour view | Day tabs、Highlights、时段行、交通段、配图与外链；生成中 liveSlots | [§16](#16-plan-plan-03--itinerary-dayhour-view) | **MVP-2** | **Done** | P0 |
| 17 | Plan | `plan-04` | Single itinerary only | 每次规划/重新规划只交付一条；无多卡短名单 | [§17](#17-plan-plan-04--single-itinerary-only) | **MVP-2** | **Done** | — |
| 18 | Plan | `plan-05` | Prefill from interests | Profile 出行兴趣可预填 Plan 偏好 chips | [§18](#18-plan-plan-05--prefill-from-interests) | **MVP-2** | **Done** | — |
| 19 | Plan | `plan-06` | Combo full options | 自定义 combo 展开始终列出全部选项 | [§19](#19-plan-plan-06--combo-full-options) | **MVP-2** | **Done** | — |
| 30 | Plan | `plan-10` | Progressive generate UX | 四段 UI：日提示 / `slot_preview` 细节提示 / 逐条 slot / pending skeleton（§11-P0） | [§30](#30-plan-plan-10--progressive-generate-ux) | **MVP-2** | **Done** | **P0** |
| 20 | Saved | `saved-01` | Saved trips grid | 仅已保存多卡；空态 | [§20](#20-saved-saved-01--saved-trips-grid) | **MVP-2** | **Done** | — |
| 21 | Saved | `saved-02` | Open saved trip | 详情 Day/Hour；无未保存 History | [§21](#21-saved-saved-02--open-saved-trip) | **MVP-2** | **Done** | — |
| 22 | Saved | `saved-03` | Unsave trip | 从详情取消收藏 | [§22](#22-saved-saved-03--unsave-trip) | **MVP-2** | **Done** | — |
| 25 | Plan | `plan-07` | Save itinerary + chat | AC1：保存行程（`messages` 可 `[]`）；AC2–3：保存含对话快照 | [§25](#25-plan-plan-07--save-itinerary--chat) | **MVP-2** · **MVP-4** | **Done**（MVP-2 AC1）/ **To-do**（MVP-4 AC2–3） | — |
| 31 | Plan | `plan-11` | Mode H prompt source | BFF 从 agent `execution=host` 拉 prompt；OPENAI_CN 执行；UI 契约不变 | [§31](#31-plan-plan-11--mode-h-prompt-source) | **MVP-3** | **Done** | **P1** |
| 33 | Plan | `plan-13` | Real transit in timeline | 消费真 navigate/directions（非估时合成 transit） | [§33](#33-plan-plan-13--real-transit-in-timeline) | **MVP-3** | **Done** | **Q4** |
| 32 | Plan | `plan-12` | Arrange OPENAI_CN stream | L2 `stream: true` + 增量 parse；首 `slot_preview` 早于整日 JSON | [§32](#32-plan-plan-12--arrange-OPENAI_CN-stream) | **MVP-3** | **Done** | **P2** |
| 23 | Chat | `chat-01` | In-page plan chat | Plan 下方唯一 Chat；BFF 本应用 OPENAI_CN 流式改当前行程（ADR-036） | [§23](#23-chat-chat-01--in-page-plan-chat) | **MVP-4** | **In progress** | — |
| 24 | Chat | `chat-02` | Local draft transcript | 回合写入 localStorage；刷新保留；登出清除 | [§24](#24-chat-chat-02--local-draft-transcript) | **MVP-4** | **Done** | — |
| 26 | Saved | `saved-04` | DB chat snapshot | 打开已保存行程可读 DB 对话；只读提示 | [§26](#26-saved-saved-04--db-chat-snapshot) | **MVP-4** | To-do | — |
| 27 | Plan | `plan-08` | Replan with confirm | 确认后换新行程（同 MVP-3 Mode H 管线）；保留 local chat + 分隔提示 | [§27](#27-plan-plan-08--replan-with-confirm) | **MVP-5** | To-do | P0·间接 |
| 28 | Plan | `plan-09` | Export PDF | 基于当前行程事实导出；不编造场所 | [§28](#28-plan-plan-09--export-pdf) | **MVP-5** | To-do | — |
| 29 | Chat | `chat-03` | Chat height resize | SE 把手仅调整高度；尊重最小高度 | [§29](#29-chat-chat-03--chat-height-resize) | **MVP-5** | To-do | — |
| 34 | Plan | `plan-14` | Boundary passthrough | BFF body 组装透传全部 `PlanBoundaries`（pace/budget/tripType/interests/must_include/timeFrom/To）到 discover + arrange | [§34](#34-plan-plan-14--boundary-passthrough) | **MVP-3r** | Done | — |
| 35 | Plan | `plan-15` | Origin geocode before enrich | ~~enrich 前 geocode~~ **Superseded** → MVP-10 plan-46 / agent F44 | [§35](#35-plan-plan-15--origin-geocode-before-enrich) | **MVP-3r** → **MVP-10** | **Superseded** | Q4 |
| 36 | Plan | `plan-16` | Keep LLM transit fields | `daySchema`/`blockSchema` 保留 transit 字段；enrich 失败显式降级；2play 侧 F42 等价校验（AC5/AC6） | [§36](#36-plan-plan-16--keep-llm-transit-fields) | **MVP-3r** | **Done** | Q4 |
| 37 | Plan | `plan-46` | MVP-10 轻骨架消费端 | 5 字段 + Travor UI + 助手 + 新 BFF 管线（make_itinerary → 逐 stop 填充） | [§37](#37-plan-plan-46--mvp-10-轻骨架消费端) | **MVP-10** | **ToDo** | P0 |
| 38 | Profile | `profile-03` | Nationality field | 注册/资料页国籍下拉（ISO alpha-3，选填）；持久化至 User；四 locale i18n | [§38](#38-profile-profile-03--nationality-field) | **MVP-11** | **ToDo** | — |
| 39 | Plan | `plan-47` | Travel advice visa slot | 出行建议页预留签证信息展示位；消费 agent `visa_requirement`（**本切片仅 spec/mock 占位，不开发查询 UI**） | [§39](#39-plan-plan-47--travel-advice-visa-slot) | **MVP-11** | **ToDo** | — |


Backlog 为 **features 1–39**。明确不在范围：SSO、双 Chat/FAB、一次多行程短名单、未保存 History、下单支付、浏览器持有 map/caller/LLM 密钥；**arrange 阶段候选池统计作主文案**（`play.plan.arrange_pool_summary` 默认隐藏，见 **30**）；搜索专名自动机翻（agent performance Q5）。



# 第二部分 — Story mapping



## 1. Header · `header-01` — App header & navigation

**用户故事 — 在主导航间切换**

作为已登录用户，我希望在「行程规划 / 我的行程 / 个人信息」之间导航，以便在不丢上下文的情况下使用各功能。

- **AC1:** 给定我已登录，当我打开任一 App 页，则顶栏导航按顺序显示：行程规划、我的行程、个人信息。
- **AC2:** 给定我在行程规划页，当页面加载，则「行程规划」为当前项。
- **AC3:** 给定我在我的行程详情页，当页面加载，则「我的行程」为当前项。
- **AC4:** 给定窄视口，当我打开移动 Menu，则仍可到达相同三项且顺序不变。

---



## 2. Header · `header-02` — Signed-in user chrome

**用户故事 — 看到当前账号**

作为已登录用户，我希望看到问候与头像，并能够登出，以便确认账号并结束会话。

- **AC1:** 给定我已登录且显示名为 Mei，当我查看 App 页，则问候区包含我的显示名。
- **AC2:** 给定我已上传头像，当我查看 App 页，则 avatar 显示圆形缩略图。
- **AC3:** 给定我已登录，当我选择登出，则会话结束并回到公开首页，且本机 `w2p.chat.`* 被清除。

---



## 3. Header · `header-03` — Locale switcher (app)

**用户故事 — 切换语言**

作为用户，我希望在 App 顶栏切换 EN/CN/HK/TW，以便界面匹配我的语言偏好。

- **AC1:** 给定我在 App 页，当我选择 CN，则用户可见文案使用简体目录（有 key 处）。
- **AC2:** 给定某 key 在所选 locale 缺失，当页面渲染，则回退且不导致页面崩溃。

---



## 4. Footer · `footer-01` — Family footer (app)

**用户故事 — App 页家族链接**

作为已登录用户，我希望在 App 页底部看到 places.family，以便打开姊妹产品。

- **AC1:** 给定我在 Plan / Saved / Profile，当页面加载，则 footer 显示 places.family 行且 where2play 为当前（非链接）。
- **AC2:** 给定我点击 what2eat 或 places.agent，当链接打开，则在新标签页打开。

---



## 5. Footer · `footer-02` — Family footer (public)

**用户故事 — 公开页家族链接**

作为访客，我希望在公开页看到 places.family，以便识别产品家族。

- **AC1:** 给定我在首页或 Auth 页，当页面加载，则存在 places.family footer（公开样式）。

---



## 6. i18n · `i18n-01` — Four-locale catalogs

**用户故事 — 文案可国际化**

作为用户，我希望所有界面文案来自 locale catalog，以便四语言一致可切换。

- **AC1:** 给定产品 UI，当检查用户可见字符串，则均通过 i18n key 解析（非硬编码唯一语言契约）。
- **AC2:** 给定 locale EN/CN/HK/TW，当切换，则导航与主要 CTA 有对应条目。
- **AC3 (plan-10):** 给定 Progressive arrange，当渲染细节提示，则 `play.plan.arrange_planning_day`、`play.plan.preview_place`、`play.plan.preview_transit`、`play.plan.preview_meal` 及 `play.plan.meal_lunch` / `play.plan.meal_afternoon_tea` / `play.plan.meal_dinner` 在四 locale 均有条目。

---



## 7. Home · `home-01` — Public landing

**用户故事 — 从首页开始**

作为访客，我希望看到产品主张并进入注册或登录，以便开始规划。

- **AC1:** 给定我打开 `/`，当页面加载，则看到品牌 logo、主标题与注册/登录入口。
- **AC2:** 给定我选择开始探索，当进入，则到达注册页。
- **AC3:** 给定我选择登录，当进入，则到达登录页。

---



## 8. Account · `account-01` — Register

**用户故事 — 创建账号**

作为访客，我希望用邮箱创建账号并可选填写兴趣，以便保存偏好并进入规划。

- **AC1:** 给定我填写必填项（姓名、邮箱、密码、确认密码），当我提交注册，则账号创建成功并可进入行程规划。
- **AC2:** 给定我留下必填项为空，当我提交，则注册不完成且标明需补字段。
- **AC3:** 给定性别、年龄、常用出发地、出行兴趣，当我注册，则这些为选填；性别**不是**必填。
- **AC4:** 给定注册页，当我查看，则有「标 * 为必填」说明，且出行兴趣标签为「出行兴趣（多选）」。
- **AC5:** 给定邮箱已被占用，当我提交，则看到邮箱字段错误且不创建重复账号。
- **AC6:** 给定密码与确认不一致，当我提交，则注册失败并提示不匹配。

---



## 9. Account · `account-02` — Sign in

**用户故事 — 登录**

作为已注册用户，我希望用邮箱密码登录，以便回到我的行程与资料。

- **AC1:** 给定有效凭证，当我登录，则进入行程规划（或会话默认 App 页）。
- **AC2:** 给定错误密码，当我登录，则看到失败提示且不建立会话。

---



## 10. Account · `account-03` — Reset password

**用户故事 — 请求重置**

作为用户，我希望通过邮箱收到重置链接，以便在忘记密码时恢复。

- **AC1:** 给定我提交已注册邮箱，当请求发送，则看到已发送说明（不泄露是否存在账号的细节以实现为准，但须有明确下一步）。

---



## 11. Account · `account-04` — Set password

**用户故事 — 设置新密码**

作为持有有效链接的用户，我希望设置新密码，以便重新登录。

- **AC1:** 给定有效重置/邀请链接，当我设置并确认新密码，则可以新密码登录。
- **AC2:** 给定过期或无效链接，当我打开设密页，则看到过期/无效状态且不能静默成功。

---



## 12. Profile · `profile-01` — User profile

**用户故事 — 维护用户资料与出行兴趣**

作为已登录用户，我希望在单页「用户资料」中编辑个人信息与出行兴趣（多选），以便规划器可使用轻量偏好。

- **AC1:** 给定我在个人信息页，当页面加载，则只有一张用户资料卡（兴趣在同一卡内，无独立兴趣卡）。
- **AC2:** 给定我修改姓名/兴趣并保存，当保存成功，则显示上次保存时间更新且刷新后仍在。
- **AC3:** 给定出行兴趣，当我多选 chips，则可选景点/美食/博物馆/公园/寺庙/夜市/购物/温泉/户外。
- **AC4:** 给定页面文案，当渲染，则不出现「轻量出行兴趣会带到规划器…」与「点选常去的玩法（可多选）。」
- **AC5:** 给定常用出发地旁，当我选择重置密码，则进入重置流程。

---



## 13. Profile · `profile-02` — Required-field markers

**用户故事 — 看清必填项**

作为用户，我希望必填字段有明确标记，以便正确保存资料。

- **AC1:** 给定个人信息表单，当页面加载，则姓名、邮箱、常用出发地标为必填（`*`），并有「带 * 为必填」类说明。
- **AC2:** 给定性别与年龄，当页面加载，则不标为必填。

---



## 14. Plan · `plan-01` — Planner form

**用户故事 — 填写边界并生成行程**

作为已登录用户，我希望填写行程边界并生成一条行程，以便看到 Day-by-Day / Hour-by-hour 草案。

- **AC1:** 给定我在行程规划页，当页面加载，则规划器为双行栅格：第一行九控件顺序为目的地 → 起始日期 → 天数 → 人数 → 每日起点 → 每日终点 → 开始 → 到 → 结束（起始日期与「节奏」同列宽/x；天数左缘与「偏好与限制」对齐）；第二行为类型/预算 | 节奏/交通 | 偏好与限制；且无「规划器」大段分区标题。
- **AC2:** 给定有效边界（至少目的地、天数、起始日期），当我选择生成行程，则中部展示**一条**行程详情（含更新日期）。
- **AC3:** 给定偏好与限制，当我操作，则可见偏好 chips 与「其他限制」自由文本（无「（可选）」后缀文案）。
- **AC4:** 给定生成进行中，当收到 `candidate_place`，则 `.plan-phase`（`.is-busy`）反映搜索态（可含已找到数量），并以同态 `.slot--candidate` 逐条展示（非 chip 条）；提交钮带 `.is-generating` 等待动效。
- **AC5:** 给定进入 arrange，When BFF 流式 NDJSON，Then 编排级行为满足 `plan-10` **AC1–AC8**（四段 UI、`slot_preview`→`slot`、禁止候选池主文案）；跨日剔除已用名称。
- **AC6:** 给定规划器，当我查看第一行，则第二项为必填「起始日期」日历控件（`type=date`，`plan-start-date`）；提交时 `startDate` 写入 PlanBoundaries，并映射为 places-agent discover 的 `bounds.start`（`bounds.end` / 各日 `date` 由起始日期 + 天数推导）。
- **AC7:** 给定中部已有行程详情，当我再次选择生成行程，则中部旧行程立即清空，再按 progressive 展示新结果（不与旧 Day/Hour 叠显）。
- **AC8 (ADR-037 / Mode H):** 给定生成请求，When BFF 处理 L2，Then **本应用 OPENAI_CN** 按天排程；**不**调用 agent `arrange_day` **execution=agent** / `plan_itinerary`。每日先 `POST /v1/arrange_day` `execution=host` 取 prompt，再 OPENAI_CN 执行（**MVP-3 Done**）。L1 仍 `discover_places`。
- **AC9:** 给定缺 `OPENAI_API_KEY`，When 生成，Then 返回明确 outcome / i18n key（非静默失败）。

---



## 15. Plan · `plan-02` — Planner validation

**用户故事 — 提交前得到清晰校验**

作为用户，我希望必填与非法输入被拦住并标出字段，以便改正后生成。

- **AC1:** 给定目的地为空，当我生成，则目的地显示错误且不调用成功规划。
- **AC2:** 给定天数为空或不在 1–14，当我生成，则天数显示错误。
- **AC3:** 给定起始日期为空或非法，当我生成，则起始日期显示错误。
- **AC4:** 给定开始与结束时间都填写且结束不晚于开始，当我生成，则时间字段显示错误。

---



## 16. Plan · `plan-03` — Itinerary day/hour view

**用户故事 — 按日与时段阅读行程**

作为用户，我希望按 Day 查看 Highlights 与 Hour 行（含交通段、缩略图、详情/地图操作），以便执行当天安排。

- **AC1:** 给定一条多日行程，当我切换 Day tab，则只显示对应日内容。
- **AC2:** 给定场所时段，当渲染，则显示时段区间、场所名、说明，并提供 **详情**（页内 place sheet，plan-46）与 **地图**（vendor 外链新标签）；legacy as-built 若仍外链详情，须在 plan-46 迁移后替换。
- **AC3:** 给定交通段，当渲染，则使用交通样式且无场所缩略图要求。
- **AC4:** 给定场所缩略图，当布局，则时间列宽与规划器「天数」列同宽约定（`--plan-col`），缩略图左缘对齐该列输入。
- **AC5:** 给定生成进行中，When progressive 揭示，Then 当日列表行为满足 `plan-10` **AC4–AC5**（逐条 `slot`、同构 `.slot--pending`）。

---



## 17. Plan · `plan-04` — Single itinerary only

**用户故事 — 一次只拿一条行程**

作为用户，我希望每次规划只得到一条行程，以便更快出结果、减少选择负担。

- **AC1:** 给定我成功生成，当结果区渲染，则不出现多条行程候选卡网格作为主交付。
- **AC2:** 给定我再次生成或重新规划成功，当完成，则中部替换为新的一条当前行程。

---



## 18. Plan · `plan-05` — Prefill from interests

**用户故事 — 兴趣带到规划器**

作为已保存出行兴趣的用户，我希望打开规划器时偏好 chips 反映我的兴趣，以便少点几次。

- **AC1:** 给定 Profile 中已选「景点、美食」，当我进入行程规划，则对应偏好 chips 为选中（可再改）。
- **AC2:** 给定我在 Plan 改 chips，当我未再保存 Profile，则不强制写回 Profile（Plan 当次边界独立）。

---



## 19. Plan · `plan-06` — Combo full options

**用户故事 — 下拉看到全部预设**

作为用户，我希望打开类型/预算/节奏/交通 combo 时看到全部预设，以便改选项而不是被当前值过滤成一项。

- **AC1:** 给定类型已填「城市漫游」，当我打开类型列表，则仍列出全部预设选项。

---



## 20. Saved · `saved-01` — Saved trips grid

**用户故事 — 浏览已保存行程**

作为用户，我希望在「我的行程」看到已保存行程卡片，以便回访。

- **AC1:** 给定我有已保存行程，当我打开我的行程，则看到多卡（标题、天数、保存时间等）。
- **AC2:** 给定我没有任何已保存行程，当我打开我的行程，则看到空态并引导去行程规划。
- **AC3:** 给定页面，当渲染，则不出现「未保存规划历史」分区。

---



## 21. Saved · `saved-02` — Open saved trip

**用户故事 — 打开已保存详情**

作为用户，我希望点击卡片查看该次保存的 Day/Hour 详情，以便回顾行程。

- **AC1:** 给定一张已保存卡，当我打开，则看到行程详情与返回「我的行程」。
- **AC2:** 给定详情页，当加载，则顶栏「我的行程」为当前项。

---



## 22. Saved · `saved-03` — Unsave trip

**用户故事 — 取消收藏**

作为用户，我希望取消收藏某行程，以便清理列表。

- **AC1:** 给定已保存详情，当我确认取消收藏，则该行程不再出现在我的行程列表。

---



## 23. Chat · `chat-01` — In-page plan chat

**用户故事 — 用页内助手改行程**

作为用户，我希望在行程规划页下方与助手对话来修改当前行程，以便不必重填整表。

- **AC1:** 给定已有当前行程，当我发送有效修改请求且 BFF 助手成功，则助手区出现回复（流式或完成后文案），且中部行程随 `itineraryPatch`（优先）或完整 `itinerary` 更新。
- **AC2:** 给定 App，当我寻找 Chat，则仅在 Plan 页内嵌入口存在（无全局 FAB 第二入口）。
- **AC3:** 给定 `POST /api/chat`，When 处理，Then BFF 直连本应用 OPENAI_CN（流式），**不**调用 places-agent `POST /v1/chat`；且助手路径**不**默认触发整单 `plan_itinerary`。
- **AC4:** 给定缺 `OPENAI_API_KEY`（或等价），When 发送，Then 返回明确 outcome / i18n key（非静默失败）；浏览器不持有 LLM key。
- **AC5:** Patch 契约：优先应用 `itineraryPatch`；若无 patch 但有完整 `itinerary`，则替换当前行程；二者皆无则仅更新对话气泡。

---



## 24. Chat · `chat-02` — Local draft transcript

**用户故事 — 对话先留在本机**

作为用户，我希望未保存前对话留在本机，以便刷新不丢；并理解清站点数据会丢。

- **AC1:** 给定我在 Plan 发送了消息，当我刷新页面，则 transcript 仍在（同一浏览器配置）。
- **AC2:** 给定我登出，当再登录，则未保存草稿不从服务器恢复（local 已清）。
- **AC3:** 给定诚实提示需求，当产品说明未保存对话，则表明仅本机、清站点数据会丢。

---



## 25. Plan · `plan-07` — Save itinerary + chat

**用户故事 — 保存行程与当时对话**

作为用户，我希望一键保存当前行程（及截至当时的对话），以便在我的行程回看。

- **AC1 (MVP-2):** 给定当前行程，当我保存成功，则我的行程出现对应卡；请求体含行程快照，`messages` 允许为空数组。
- **AC2 (MVP-4):** 给定当前行程与若干 chat 消息，当我保存成功，则 DB 含截至当时的对话快照。
- **AC3 (MVP-4):** 给定保存后我又继续聊天，当我未再次保存，则 DB 快照仍为上次保存点；local 为更新真源。

---



## 26. Saved · `saved-04` — DB chat snapshot

**用户故事 — 回看保存时的对话**

作为用户，我希望在已保存详情中阅读保存当时的对话，以便回忆为何如此安排。

- **AC1:** 给定保存时含对话，当我打开该行程详情，则展示只读对话且注明来自数据库快照。
- **AC2:** 给定只读对话区，当展示，则说明续聊需回到规划并再次保存。

---



## 27. Plan · `plan-08` — Replan with confirm

**用户故事 — 确认后重新规划**

作为用户，我希望在丢弃未保存行程前得到确认，并在重新规划后保留本机对话与分隔提示，以便不丢聊过的约束语境。

- **AC1:** 给定我选择重新规划，当对话框出现，则说明将删除当前未保存行程并生成新行程，且本机对话会保留并加分隔。
- **AC2:** 给定我取消，当关闭对话框，则当前行程与对话不变。
- **AC3:** 给定我确认，当重新规划成功，则中部为新行程；local chat 保留并出现系统分隔；已保存库中旧记录不受影响。
- **AC4:** 给定 replan 请求，当发送，则携带截断后的 chat 上下文；L2 走 **MVP-3** 管线（Mode H host prompt + OPENAI_CN），不默认 agent `arrange_day` execution=agent。
- **AC5 (plan-10):** 给定我确认重新规划，When BFF 流式返回，Then 中部清空后满足 `plan-10` **AC1–AC5**（同 progressive 事件契约）。

---



## 28. Plan · `plan-09` — Export PDF

**用户故事 — 导出行程 PDF**

作为用户，我希望导出当前行程 PDF，以便离线分享或打印。

- **AC1:** 给定当前有行程，当我导出 PDF，则文件内容基于当前行程事实与文案。
- **AC2:** 给定某场所字段缺失，当导出，则使用占位/省略，不编造场所事实。

---



## 29. Chat · `chat-03` — Chat height resize

**用户故事 — 调高聊天区**

作为用户，我希望拖动把手只调整聊天高度，以便多看对话而不挡行程。

- **AC1:** 给定 Plan Chat，当我拖动高度把手，则面板高度变化且不低于最小高度。
- **AC2:** 给定 `prefers-reduced-motion: reduce`，当使用页面，则飞行动画关闭；resize 仍可用。

---



## 30. Plan · `plan-10` — Progressive generate UX

**用户故事 — 逐步看到正在生成什么**

作为用户，我希望生成行程时按条揭示（景点/交通/餐）并看到下一条加载中，以便等待时知道进度。

**规格真源：** [itinerary-design.md](./itinerary-design.md) · [performance.md](../agent-specs/performance.md) 第十一节 §11-P0 · 完工：**Done（单测/契约）/ E2E 待签**（切日 tab 见 AC8 **部分**）

- **AC1:** 给定生成进入 arrange，当页面展示，则**行程日提示**仍为「正在安排第 d/N 天…」（`.plan-phase.is-busy`）。
- **AC2:** 给定尚无 `slot_preview`，当 LLM 等待，则**行程细节提示**为 `play.plan.arrange_planning_day`（**不**以 `play.plan.arrange_pool_summary` 作主文案）。
- **AC3:** 给定 `slot_preview.kind=place|transit|meal`，当渲染，则分别使用 `play.plan.preview_place` / `play.plan.preview_transit` / `play.plan.preview_meal`（及 `play.plan.meal_`* 餐段）插值 name/reason/window。
- **AC4:** 给定一日多站，当 BFF staged emit，则**行程**列表一次只多一条 `.slot`（`place` 为兼容 alias）；下条前**加载中提示**为 `plan-slot-pending` 同构 skeleton（非虚线框）。
- **AC5:** 给定 BFF 发 `slot_preview` 后立即发 `slot`，When UI 同帧收到多条，Then reveal 队列仍逐条展示（不整日同 tick 刷屏）。
- **AC6:** 给定 `prefers-reduced-motion: reduce`，当生成，则无 pending shimmer；BFF stage 间隔可由 `PLAN_SLOT_STAGE_MS` 配置为 0。
- **AC7:** 给定 progressive `slot` 与 `day_done`，When 比对 `itinerary.days[].slots`，Then 二者来自同一 `expandArrangeDayToSlots`（含首尾/站间 transit），避免最终跳变。
- **AC8:** 给定单日事件序，When BFF emit，Then 为 `arrange_day_start` → `day_highlights` → (`slot_preview` → `slot`) → `day_done`；`discover_done` **不**驱动 arrange 主文案；`day_done` 后自动聚焦下一 Day tab（未排日 `play.plan.day_n_queued`）。

---



## 31. Plan · `plan-11` — Mode H prompt source

**用户故事 — 排程 prompt 与 agent 同源**

作为产品，我希望 BFF 从 places-agent `execution=host` 拉取排程 prompt，再用本应用 OPENAI_CN 完成排程，以便与 MCP / HTTP 共用一套拼装，并稳定纳入地标与交通约束。

**规格：** [itinerary-design.md](./itinerary-design.md) §5.1 / §9 · agent Feature **35**（**Done**）· **MVP-3**

- **AC1:** 给定生成 L2，When 每日排程，Then BFF `POST /v1/arrange_day` `execution=host`，使用返回的 `system_prompt` / `user_prompt` / `output_contract` / `candidates_slim`；**不**再默认本地 duplicate prompt；**不**调 agent 侧 LLM（execution=agent）。
- **AC2:** 给定切换 prompt 来源，When UI 收事件，Then 仍为 `slot_preview` → `slot` → `day_done` 契约（不变）。
- **AC3（地标）:** 给定 test-plan §8 探针目的地（如 London / 西安 seed 城），When 完整生成成功，Then 行程 slots 至少包含一处该城 canonical must-see（与 ADR-038 discover 种子一致；不得仅靠 LLM 幻觉）。
- **AC4（交通偏好）:** 给定用户在 Plan 表单选择「交通」偏好（如捷运/步行），When 调用 host arrange，Then 请求体携带对应约束字段（与 agent `buildSchedulePrompt` 契约一致）。

---



## 32. Plan · `plan-12` — Arrange OPENAI_CN stream

**用户故事 — 首站更早出现**

作为用户，我希望不必等整日 JSON 完成才看到第一条站点预告。

**规格：** [itinerary-design.md](./itinerary-design.md) §5.2 / §10 · [performance.md](../agent-specs/performance.md) §11-P2 · **W2c**

- **AC1:** 给定 arrange OPENAI_CN `stream: true`，When 解析出首个 block，Then 在整日 JSON 完成前即可发出首个 `slot_preview`。
- **AC2:** 给定流式失败/超时时，When 处理，Then 映射 `errors.arrange_timeout` 或等价 i18n，且不留下半截不可用日为「成功」。

---



## 33. Plan · `plan-13` — Real transit in timeline

**用户故事 — 交通段用真实耗时与方式**

作为用户，我希望站间交通显示真实 directions/navigate 耗时与方式，并反映我在规划器选择的交通偏好，而不仅是统一估时文案。

**规格：** agent [performance.md](../agent-specs/performance.md) §0.1 Q4 · Feature **37**（**Done**：`legs_to_here` / `transit_outcome`）· **MVP-3**

- **AC1:** 给定相邻两站有坐标且 agent 已 enrichment，When 行程含 transit 行，Then 时长/方式来自 navigate 或 directions 结果（密钥不进浏览器）。
- **AC2:** 给定 directions 失败，When 降级，Then 仍可展示行程其余站，并有明确失败/估时回退提示（i18n key）。
- **AC3:** 给定 LLM block 含 `duration_min` 与 agent `legs_to_here`，When `expandArrangeDayToSlots` 映射，Then 场所时段使用该时长（**禁止**一律默认 ~15min 占位）；transit 行展示具体方式（如步行/地铁/打车），非泛化「前往下一站」。

---



# 附录 — 与 mock / 产品规格对照


| 主题                   | 真源                                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| 视觉与 DOM              | [ui-mockup/](./ui-mockup/) + [2play-design.md](./2play-design.md) §1/§3                                               |
| 产品边界                 | [`product-backlog.md`](../product-backlog.md) §5                                                                            |
| 兴趣标签文案               | 「出行兴趣（多选）」；无两段已删说明                                                                                                        |
| 性别                   | 注册/资料均非必填                                                                                                                 |
| Chat 真源              | 草稿 local；保存时 DB                                                                                                           |
| 主路径                  | 每次一条行程                                                                                                                    |
| Progressive / NDJSON | [itinerary-design.md](./itinerary-design.md) · feature **30–32**；冲突时 **itinerary-design + plan-10** > `2play-design` 摘要 |
| 四段 UI 命名             | 行程日提示 / 行程细节提示 / 行程 / 加载中提示（术语表）                                                                                          |


**下一步：** **W2r** MVP-3r 收尾（**35** 并入 MVP-10）→ **W2.5** **MVP-10 plan-46**（P4，依赖 agent F44）→ **W3** MVP-4 余 story → **W4** MVP-5。实现 UI 以 `06-plan*.html` + `2play-design.md` §4.7 + `itinerary-design.md` §16–17 为对齐标准。

---



# 34 — Plan — plan-14 — Boundary passthrough

**类别：** Plan · **MVP-3r** · Feature **34** · 完工：**Done**（2026-08-24：natural_language 组装、时间补零三处、首块时间硬校验重试；vitest 153 绿 + live probe 首块 09:30 确认）

**作为** where2play 用户  
**我希望** 我填的边界条件（节奏、预算、出行类型、兴趣、每日起止时间、人数）完整进入 arrange 排程  
**以便** 行程符合我的输入，而不是被丢掉后按默认排

**实现前修订（2026-08-24，契约调研后）：**

1. `PlanBoundaries`（`src/core/itinerary-types.ts`）无 `must_include` 字段且 UI 无输入口（属 ChatBox MCP 概念）→ 从本 story 剥离；新增必去输入属新产品需求另行立项。
2. agent `discover_places` schema 只认 `city/bounds/origin/numDays/providers/locale`，不接受偏好 → AC 收敛为 arrange body 全量透传；discover 侧改善走 `plan-15`（带坐标 origin 改善搜索锚点）。
3. agent 死字段规避：`preferences.interests` agent 接受但不进 prompt；NDJSON 路径丢 `party_size`/`num_days` → 2play 一律把 tripType/interests/constraints 拼进 `preferences.natural_language`（agent `buildUserMessage` 已消费）。
4. 时间补零缺陷（三处）：`plan-validate.ts` 字符串比较、`plan-arrange-llm.ts` regex 允许 `\d{1,2}`、`itinerary-map.ts` `addMinutes` 要求严格 `\d{2}` → 随本 story 一并修。

## AC

- **AC1:** 给定 `PlanBoundaries`（pace/budget/tripType/interests/constraints/timeFrom/timeTo/partySize），When BFF 组装 `arrange_day`（host）请求体，Then 已有字段全部透传（pace/budget/party_size/preferences.time_from/time_to/transit_preferred 保持），tripType/interests/constraints 拼入 `preferences.natural_language`，无一静默丢弃。
- **AC2:** 给定时间输入 `"9:00"`/`"10:00"`（非补零），When 校验与解析，Then 归一为 `HH:MM` 后比较与传递，不再误判顺序、不再因单数字小时解析失败。
- **AC3:** 给定用户填每日起止 9:30/20:00，When 2play LLM 生成日卡，Then 首 block start_time 与 `timeFrom` 一致（±5min；user_prompt 带硬规则，覆盖「早于」与「无视默认 10:00 漂移」两个方向；违规触发一次纠偏重试）；末 block 结束不晚于 `timeTo`（交通段除外）。
- **AC4:** 透传与时间归一有契约/单元测试覆盖（body 断言 `natural_language` 含 tripType/interests/constraints；补零三处各有用例），非仅手测。

# 35 — Plan — plan-15 — Origin geocode before enrich

**类别：** Plan · **MVP-3r** · Q4 · Feature **35** · 完工：**Superseded**（2026-08-31：并入 MVP-10 plan-46 / agent Feature 44 起点 geocode；MVP-3r 不再独立交付）

~~**作为** where2play 用户~~  
~~**我希望** 每日酒店/终点先解析成坐标再请求真实交通~~

**迁移：** 起点作为 Stay stop + `plan_next_stop` 内 geocode；AC 见 Feature **37** plan-46 与 agent Feature **44**。

# 36 — Plan — plan-16 — Keep LLM transit fields

**类别：** Plan · **MVP-3r** · Q4 · Feature **36** · 完工：**Done**（2026-08-24：`blockSchema`/`daySchema` 保留 `legs_to_here`/`from_origin`/`to_destination`/`transit_outcome`；enrich 失败显式 `transit_outcome: "partial"` → i18n `play.plan.transit_estimated`；2play 侧站间时序校验 `stationTimingViolation` + 同日餐厅去重 `sameDayRestaurantDedupViolation` 接入重试回路；vitest 168 绿，tsc 0 错误）

**作为** where2play 用户  
**我希望** LLM 输出的交通字段被保留、enrich 失败可见  
**以便** 站间不再一律显示默认步行 10+ 分钟且无方式说明

## AC

- **AC1:** 给定 arrange LLM 输出含 `from_origin`/`to_destination`/`legs_to_here`，When Zod 解析，Then schema 保留这些字段（不再剥离），进入 `expandArrangeDayToSlots`。
- **AC2:** 给定 agent `enrich_arrange_transit` 失败或降级（`transit_outcome: "heuristic" | "partial"`），When 展示，Then UI 标注降级原因（i18n key），不静默当作成功。
- **AC3:** 给定 enrich 成功，Then 站间 transit 用 `legs_to_here` 真实时长/方式替换默认估时；`estimateTransferMin` 仅在无任何数据时兜底。
- **AC4:** 契约测试：schema 快照含 transit 字段；enrich 失败路径有显式断言（非 catch 后吞）。
- **AC5 (F42 等价，2play 侧站间时序):** 给定 arrange LLM 输出 blocks 含 `legs_to_here`，When `streamArrangeDay` 解析后，Then 校验 block[i].start_time ≥ block[i-1].end_time + recommended_leg.duration_min − 5min 容差；违规触发一次重试（带 error message），重试仍违规则硬失败并标注降级原因。
- **AC6 (F42 等价，2play 侧同日餐厅去重):** 给定 arrange LLM 输出含两个 meal 块（lunch/dinner）同名，When `streamArrangeDay` 解析后，Then 校验失败并触发一次重试；重试仍违规则硬失败。

---

# 37 — Plan — plan-46 — MVP-10 轻骨架消费端

**类别：** Plan · **MVP-10** · Feature **37** · 完工：**ToDo**（2026-08-31 方案确定；2026-09-02 mock/spec 锁定；BFF 部分落地；**UI 须 100% 对齐 mock 后签收**）

**作为** where2play 用户  
**我希望** 填 5 个必填项后由悬浮助手补全偏好，并看到骨架顺序再逐站填充的真实行程  
**以便** 首站更快可见、交通与景点信息更准确

**规格：** [itinerary-design.md §16–17](./itinerary-design.md) · [2play-design.md §4.2.1 / §4.7 / §3.9 / §4.9](./2play-design.md) · mock [`ui-mockup/`](./ui-mockup/) · agent [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md) 批次 **11 + 16 + 18** · [ADR-046](../adr/ADR-046-trip-store-pg-memory-fetch.md) · 依赖 agent **44**（Done）+ **63/64**（Trip Store / fetch）+ **65**（Done，无 display）+ **75–77**（ToDo）

**禁止（回归门禁）：** Plan 页不得保留 MVP-2/3 **双行 `plan-board`**（类型/节奏/交通/chips/限制/时段）；「规划行程」不得直接 POST 生成；不得用页内固定 `PlanChatPanel` 代替 `plan-nav` 做 intake；不得在未渲染 `plan-travel-tips` / `plan-constraints` 时标 Done。

## AC

### UI — 全站 Travor 与公开页

- **AC0:** 给定任意 App 或 Auth 路由，When 渲染，Then `body` 含 `data-style="travor"` + Travor token（§4.7）；结构类与 mock 画廊 `01–10` 一致（冲突以 mock 为准）。
- **AC0b:** 给定注册页（`02-register.html`）或资料页（`07-profile.html`），When 渲染，Then `.register-card__grid` 含右侧 **头像 bowl**（可选；`data-testid="field-photo"`），与 §3.2/§3.4 一致。
- **AC0c:** 给定 Home（`01-home.html`），When 渲染，Then CTA 行外另有 **注册链**（`auth-links` → `/register`）。

### UI — 起飞与助手

- **AC1:** 给定 Plan 页，When 用户只填目的地/起始日期/天数/人数/预算并点「规划行程」，Then 调起悬浮 **`plan-nav` 助手**（不直接生成）；仅 **`plan-takeoff` 单行 5 格**，无 legacy 双行字段；5 字段 i18n key，无硬编码英文。
- **AC2:** 给定助手展开，When 8 步问答 b–h（[`performance.md §12.11`](../agent-specs/performance.md) / §4.2），Then 任一步可「终止」或「使用默认」跳过；`plan-nav__context` 进度摘要可见。
- **AC3:** 给定助手，When 拖动左上角拉手，Then 面板可 resize（最小=默认 27×45rem）；`prefers-reduced-motion` 关 shimmer。

### UI — Travor 与行程列表

- **AC4:** 给定 `data-style="travor"`，When 渲染 Plan，Then 暖底 `#FAF7F1`、浅橙实心按钮、助手头/用户气泡 `#FFE3D3`、标签 teal `#068A7F`（见 §4.7 token 表）。
- **AC5:** 给定填充进行中，When 渲染首日，Then **起点为 Stay 标准 stop 卡片**（`data-testid="stop-origin"`），非仅 transit 内嵌酒店名。起点 slot 的 `nativeId` / 坐标 / `photoUrl` 来自**同一张** intake `originStay` 卡（[ADR-053](../adr/ADR-053-origin-stay-as-stop-card.md)）；例：凯悦逸扉（西安钟楼回民街店）**不得**绑西安钟楼。
- **AC6:** 给定相邻两站已上屏，When 渲染 transit，Then 单行 `从 · {A} 前往 · {B}：[mode|dur|cost] / [mode|dur|cost]`；place 名加粗；降级显式 i18n（§17.2）。
- **AC7:** 给定 panel 头，When 行程生成中或完成，Then 「重新规划 / 保存行程 / 导出 PDF」在 `panel__head-actions` 右侧；无底部 sticky 操作条。

### UI — 出行限制与小贴士（mock：`plan-constraints` / `plan-travel-tips`）

- **AC12:** 给定助手接管（点击「规划行程」）后，When Plan 主区渲染，Then **隐藏** `plan-takeoff`，显示只读 **出行限制** panel（`data-testid="plan-constraints"`）按 [`ui-mockup/06-plan-qa.html`](./ui-mockup/06-plan-qa.html)：桌面 **共享四列轨**；起飞 8 项两行（目的地/行程开始日期/行程天数/出行人数，行程类型/行程预算/动线节奏/交通偏好）+ 横线下 intake **一行** 3 项（每日起点、每日出发时间、其他要求 span 2）；**无必去点行**；标签与值同行左对齐（同字号）；未答 intake 为 `play.plan.constraint_pending`；节奏/交通显示 i18n 目录词。Intake 每答一题，对应格立即更新。
- **AC13:** 给定 `make_itinerary` 已写入骨架且 `travel_tips` 已写 `artifacts`，When Plan 主区为 `planning`/`done`，Then **出行小贴士** panel（`data-testid="plan-travel-tips"`）可见；四卡来自 **`fetch_trip_details` `artifacts`**（NDJSON `tips` 仅为进度信号）。**intake 期间不展示**贴士面板。
- **AC13b:** 给定步骤 g 芯片与贴士 01，When 渲染，Then 芯片来自 fetch **`candidates` 上 `must_see` 名**；贴士 01 来自 fetch **`artifacts.tips.iconic_places`**（make 之后 grounded）。**禁止** intake 期 ungrounded `travel_tips` 当芯片；**禁止**把 discover HTTP 包络当 UI 真源。
- **AC13c:** ~~给定点击「规划行程」，When 助手仍在 b–f，Then BFF 已开始 `POST /api/plan/discover`~~ **废止（Feature 41 Story 1）**：CTA 后、步骤 g 前 **不** discover。芯片改到 Feature 41 Story 3。仍禁止为展示调 tips-prose / OPENAI_CN。
- **AC13d:** 给定问答结束，When `POST /api/plan`，Then body 带提前 discover 的 `trip_id`/`revision`；编排 **不**在有池时二次 discover；`travel_tips` 在 fetch 骨架之后。
- **AC24 (MVP-18 F71):** 给定新会话或重新规划，When 渲染起飞栏预算，Then 默认选中 `mid`（适中）；约束条/助手顶栏显示 i18n「适中」，**不**显示英文 `comfort`/`mid`。
- **AC25 (MVP-18 F77):** 给定助手步骤 c 用户输入 `7:00 am` 或「早上七点」，When 开始填站，Then `time_from` 为 `07:00`；无法解析则 `09:00`，**不**出现因口语时刻导致的 `play.errors.invalid_input`。
- **AC26 (MVP-18 F72):** 给定 fetch 到的 skeleton 每天首位为同一酒店 stay，When 助手骨架预览，Then 每天突出景点/餐饮；相同酒店收成「从 {酒店} 出发」；亮点主标题不与副标题重复同一 `day_theme` 字符串。
- **AC27 (MVP-18 F75):** 给定 `make_itinerary` 或 `plan_next_stop` 刚成功，When 更新预览或行程卡，Then 数据来自随后的 `fetch_trip_details`（`skeleton` / `filled`）；NDJSON 仅进度。
- **AC28 (MVP-19):** 给定步骤 g，When 池上 8 处 `must_see` 且用户指定 3 处，Then 芯片仍为热门 `must_see`；`mustInclude` 仅为用户 3 处；**禁止**把芯片收成 3 处或让 BFF 把池标改成 3。
- **AC29 (MVP-19 / P0b):** 给定 make 非 200 且 `trip_id` 存在，When BFF 处理，Then 先 fetch skeleton；合法多站则续 fill；否则不进入 filling。失败文案：仅 abort/超时 outcome → `play.plan.phase_make_timeout`；其它（含 LLM 403 / validation）→ `errors.make_itinerary_failed`（UI 映射 `assistant_make_failed`），**禁止**把非超时失败一律显示为「框架超时」。
- **AC30 (MVP-19):** 给定 fetch 骨架成功，When 助手，Then 先出现 `assistant_know_enough`，再出现按日站名（每日非 stay ≥1 可见），再 `assistant_skeleton_ready`，**然后**才循环 fill。
- **AC31 (MVP-19):** 给定 fill 进行中，When 主区，Then `plan-phase` + `plan-slot-preview` + 未填 `skeleton-stop`；不得只显示酒店一站。
- **AC32 (MVP-19 / 24-P0-ui-A):** 给定一站 `plan_next_stop` 成功，When 助手与主区，Then 主区 +1 slot；助手 **覆盖** 全日唯一进行中进度行（与 `.plan-slot-preview` 同句，含 transit 信息时用 `preview_transit` / `assistant_filling_stop` 之一），**禁止**堆叠多条 preview；随后 fetch 只升 revision。
- **AC33 (MVP-19):** 给定全部 fill 完成，When 助手，Then `assistant_plan_complete` 含目的地、天数、人数、行程类型；**无**残留 fill 进行中行。
- **AC34 (MVP-19):** 给定规划 `done`，When 写入 `PlanSessionCache`，Then 含 `trip_id`/`revision`；`GET /api/plan/current` 可再 fetch。
- **AC35 (24-P0-ui-A):** 给定 `slot_preview` place/transit/meal，When 渲染助手或主区预览，Then 文案为无「入选原因 / 选择原因 / 推荐原因」的 `preview_*` 模板；站类型用 `kind_*` / `meal_slot_*` i18n，**禁止**裸露 `STAY`/`ATTRACTION`/`MEAL`。
- **AC36 (24-P0-ui-A):** 给定 discover / make / fill 任一 agent 流进行中，When 助手 composer，Then `plan-nav-input` 与 `plan-nav-send` disabled（`aria-disabled`）；完成后或失败解锁。
- **AC37 (24-P0-ui-B):** 给定骨架已生成，When 助手 `plan-thread-skeleton`，Then 渲染 route-spine（日主题 + 站珠：起点/景点/餐+店名）；**无**出发文案、**无**交通芯片、**无**到/停；thread **无**白底 agent 卡。
- **AC38 (24-P0-ui-B):** 给定 fill 进行中或完成且有 `filled` 切片，When 助手 `plan-thread-fill-timeline`，Then 同 spine；站间为 `{time} 出发前往下一站` + 模式耗时；站上有到站与停留；散文进度仍全日一条覆盖，**不**用纯文字时间线堆叠。
- **AC39 (24-P0-ui-C):** 给定已填 place/stay/meal stop，When 渲染主区 `.slot-thumb`，Then 缩略图为 **1:1**；有供应商 `photoUrl` 时显示图，无则空占位（禁止硬编码城景图）。stay 拇指 = 账本卡 `photos[0]`（与详情同源）。
- **AC40 (24-P0-ui-C):** 给定用户点击缩略图或 **详情**，When 交互，Then 打开 `place-sheet`（Esc / backdrop / 关闭可关）；**地图**仍新标签打开 `mapUrl`。
- **AC40b (lightbox ×2):** 给定 place-sheet 有可展示图，When 用户点击图打开 lightbox（`data-testid="place-photo-lightbox"`），Then lightbox `<img>` **同一** `photos[0]` / `photoUrl`（agent `maxWidthPx=800` 解析）；不另开第二真源或 2play Photo 代理。视觉固有宽度约为原 400 解析的 ×2。
- **AC41 (24-P0-ui-C):** 给定 fill `done`，When 主区当日列表，Then **无**日底骨架清单、**无**残留 `.plan-slot-preview`（含 transit 进行中文案）；仅已填 slot（+ transit 行）。
- **AC42 (ADR-052):** 给定 Plan discover / iconic / intake 酒店搜，When BFF 调 agent，Then **省略** `providers[]`（交给 agent 区域自动选）；**禁止** `providersForDestinationText` 汉字→`["AMAP","GOOGLE_MAPS"]`；`providersForPin` 对大陆不得硬传双源。Agent discover **不得**再扩双源（Feature **89**）；酒店 / stay 与景点同一套 D2+D4。
- **AC43 (ADR-052 D9/D10):** 给定大陆已填景点（如杭州植物园），When 主区列表，Then 名称/地址为槽位中文，`provider` 为 `AMAP`（除非该站 D4）。When 打开 place sheet，Then `get_place_details` 使用槽位 `provider`+`nativeId`+UI locale；标题保持 CJK，**禁止**半秒后换成 Google 英文名。

### BFF — 新管线
- **AC14:** 给定填充阶段，When 渲染，Then `plan-phase` 左对齐 meta（如 `骨架 HH:MM · 填充中`）+ 进度文案；主列表含已填充 `.slot`、`.slot--transit`、pending `.skeleton-stop.is-pending`。

### UI — Stop 行：详情 / 地图 / 场所浮层

- **AC15:** 给定非 transit 的 place/stay/meal stop，When 渲染 `.slot-actions`，Then 显示 **详情**（`data-testid="stop-detail-open"`）与 **地图**（`data-testid="stop-map-open"`）两项；`.slot--transit` **无** 详情/地图按钮。
- **AC16:** 给定用户点击 **详情**，When 交互，Then 打开页内 **place sheet** modal（`data-testid="place-sheet"`，`role="dialog"`）；**不**跳新标签；Escape /  backdrop / 关闭钮可关；焦点 trap 至 dialog。
- **AC17:** 给定 place sheet 打开，When 渲染，Then 含：**场所事实**（图/名/评分/kind/地址/电话或开放/价格/source）、**本行程安排**（Day + 时段 + slot 摘要）、**如何到达**（复用当日 `legs_to_here` 推荐与备选，与列表 transit 行一致）；底部 **在地图中打开**（`data-testid="place-sheet-map"`，新标签 vendor URL）。
- **AC18:** 给定用户点击 **地图**（列表行），When 交互，Then 直接新开标签打开 vendor 地图 URL（与 place sheet 内「在地图中打开」同源）；URL **不含** API key query。
- **AC19:** 给定 BFF 拉取详情，When place sheet 需 enrich，Then BFF → agent `get_place_details`（**仅** `slot.provider` + `slot.nativeId` + UI locale；或 `fetch_trip_details` fields 已含富信息则不再二次请求）；加载/失败态 i18n（`play.plan.place_sheet_*`）；缺失字段显式「不可用」非编造。槽位已是 CJK 名时，拉丁文详情名不得覆盖（ADR-052 D9/D10）。

### UI — 已保存详情同构（`09-saved-detail.html`）

- **AC20:** 给定已保存详情页，When 渲染行程块，Then 与 Plan 完成态 **同构**：constraints + travel tips + day tabs + stay/transit/slot 列表 + 详情/地图/place sheet；panel 头为「返回列表 / 导出 PDF / 取消收藏」。
- **AC21:** 给定 MVP-4 `saved-04` 已交付，When 打开含对话快照的保存项，Then 在行程块 **下方** 追加只读 DB 对话区（`data-testid="chat-transcript"`）；**MVP-10  mock 可无此块**，不阻塞 plan-46 签收。

### BFF — 新管线

- **AC8:** 给定助手问答完成且提前 discover 已有 `trip_id`，When BFF 规划，Then `make_itinerary`（带池）→ `travel_tips`（骨架后）→ 循环 **`plan_next_stop`**，**每步写成功后** **`fetch_trip_details`**；**不**调用 `display_current_stop`；**不**再调本地 OPENAI_CN arrange。
- **AC8c (MVP-18):** 给定写工具返回 `trip_id`+`revision`，When UI 需要骨架、已填站、artifacts 或约束，Then **必须** fetch 对应 `fields[]`；禁止把生成工具响应或 BFF LLM 散文当作第二真相源。
- **AC8b:** 给定 agent 尚未提供 `trip_id`（过渡双写期），When BFF 仍收到旧 `next_tool_call` 形态，Then 可临时兼容；**正式签收**以 `trip_id` 路径为准（与 agent MVP-16 P1 对齐）。
- **AC9:** 给定 NDJSON 流，When 客户端消费，Then 处理 `skeleton_start`/`skeleton_day`/`skeleton_done`/`stop_filled`/`day_done`/`itinerary_done`（§16.2）；废弃 staged sleep 假 progressive。
- **AC10:** 给定 agent 返回 `transit_outcome: partial|heuristic`，When UI 展示，Then 显式降级文案（i18n key），不静默成功。

### 与 agent 30 城 E2E 对齐（parity）

- **AC22:** 给定 agent [`e2e-test-result/INDEX.md`](../agent-specs/e2e-test-result/INDEX.md) 中 **27/30 通过** 场景的输入边界（城市/天数/节奏/预算/酒店/必去），When 经 where2play Plan 页完整走通（助手默认值 + 同边界），Then 到达 `trip_complete`；骨架日主题与 stop **名称集合** 与对应 agent markdown **一致**（允许 vendor 侧评分/图差异；时段 ±15min）。
- **AC23:** 给定 agent 标记 **失败** 的三城（Paris #2 / Tokyo #3 / Hanoi #21），When where2play 复现，Then 失败原因同类（`make_itinerary` 超时或校验失败）；UI 展示可读 `data.detail`；**不**用 fixture 冒充成功。

### 性能（Lisbon 4D live 基线）

- **AC11:** 给定 Lisbon 4D fixture/live，When 完整规划，Then 首 stop 可见 < 30s；总墙钟 < 90s（与 agent §12.9 探针一致，允许 ±15% 环境方差）。

### 明确排除

- PDF 导出实现属 MVP-5；本 story 仅保留按钮占位或 disabled + i18n「即将推出」。
- **MVP-4 改行程 Chat** 不在本 story；`plan-nav` 助手 **仅 intake**（8 步边界收集），非 post-plan 编辑。
- Legacy Mode H progressive（候选池面板、`plan-board` 双行、页内 `PlanChatPanel` 直连 `/api/chat` 作 intake）须移除或 feature-flag 隔离，不得与 MVP-10 并存于默认路径。

---

# 38 — Profile — profile-03 — Nationality field

**类别：** Profile · **MVP-11** · Feature **38** · 完工：**ToDo**（2026-09-01 规格确定）

**作为** 已登录用户  
**我希望** 在注册和个人资料中选择我的国籍（护照签发国）  
**以便** 后续出行建议页能按我的护照查询目的地签证要求

**规格：** [ADR-044](../adr/ADR-044-orizn-visa-rest-adapter.md) D4 · [2play-design.md](./2play-design.md) §3.2/§3.4 · agent Feature **48**（**Done**）· 开放清单 [`04-rome.md` 开发计划](../agent-specs/e2e-test-result/04-rome.md)

## AC

- **AC1:** 给定注册页或个人信息页，When 页面加载，Then 显示「国籍」下拉（`play.register.nationality` / `play.profile.nationality`），位于性别/年龄行下方或同行扩展行；**选填**（不标 `*`）。
- **AC2:** 给定下拉选项，When 渲染，Then 值为 ISO 3166-1 alpha-3（如 `CHN`、`USA`、`JPN`）；展示名通过 `Intl.DisplayNames` 按当前 locale 本地化（**不在源码内嵌单一语言国家名大表**，ADR-044 D4）。
- **AC3:** 给定首项，When 用户未选择，Then 默认「请选择 / Prefer not to say」空值（`nationality` 存 `null`）。
- **AC4:** 给定注册提交含 `nationality: "CHN"`，When 账号创建成功，Then DB `User.nationality = "CHN"`；刷新资料页仍为 `CHN`。
- **AC5:** 给定资料页修改国籍并保存，When 保存成功，Then `User.nationality` 更新；四 locale 均有对应 i18n key。
- **AC6:** 给定 API 校验，When 传入非法 alpha-3，Then 返回字段错误（`play.errors.nationality_invalid`），不写入 DB。

**明确排除（本 story）：** 不在 Plan 页展示签证查询；不调用 agent `visa_requirement`（属 Feature **39** 后续实现）。

---

# 39 — Plan — plan-47 — Travel advice visa slot

**类别：** Plan · **MVP-11** · Feature **39** · 完工：**ToDo**（2026-09-01 规格占位）

**作为** 产品  
**我希望** 在规格与 mock 中预留「出行建议页」的签证信息展示位  
**以便** 后续切片可接入 agent `visa_requirement`，而无需返工 Profile 契约

**规格：** [2play-design.md](./2play-design.md) §3.5.6 · agent Feature **48**（**Done**）· [`refactor-plan-archive.md`](../knowledge/agent/refactor-plan-archive.md) · [`04-rome.md` 开发计划](../agent-specs/e2e-test-result/04-rome.md)

## AC（本切片 = spec + mock 占位，**不开发**运行时查询）

- **AC1:** 给定 [2play-design.md](./2play-design.md)，When 阅读 §3.5.6，Then 描述出行建议页签证区块：输入 = `User.nationality` + 目的地 alpha-3；**写** = BFF → agent `POST /v1/visa_requirement` 入 `artifacts.visa`；**展示** = `fetch_trip_details`；字段含 requirement、免签天数、材料摘要、`last_verified`、官方来源链接。
- **AC2:** 给定 [ui-mockup/](./ui-mockup/)，When 新增或标注占位页（如 `10-travel-advice.html` 或在 design 文档 wireframe），Then 含 `.visa-advice` 区块与 i18n key 列表（`play.travel_advice.visa_*`）。
- **AC3:** 给定 honesty 要求，When 规格描述配额/降级，Then 明确 Orizn 配额耗尽时显示 i18n 降级态（非编造签证事实）；无 nationality 时提示用户至资料页补充。

**实现切片（后续立项，不在 MVP-11 spec 范围）：** BFF `/api/travel-advice/visa` + 真实 UI 渲染。

---

# 40 — Plan — plan-48 — 助手叙事 + 同流 fetch（MVP-19）

**类别：** Plan · Feature **40** · 完工：**Done**  
**规格：** [2play-design.md](./2play-design.md) §4.10–§4.11 · agent F78–F82  
**依赖：** Feature 37 AC28–AC34；agent 批次 19

**作为** 规划用户  
**我希望** 每一步都知道系统在搜点、排骨架还是填站，并先看到完整骨架再出细节  
**以便** make 慢或 fill 未完时也不会以为每天只有酒店

## AC

- **AC1:** 给定 CTA，When 助手打开，Then thread 可有 `assistant_discovering`；g 门闩 fetch candidates。**CTA 当秒 discover 已被 Feature 41 Story 1 废止**（intake 未完成前不搜点、约束条必去为 `—`）。
- **AC2:** 给定 intake 结束，When 规划开始，Then 固定句 `assistant_know_enough`，主区 `phase_making`。
- **AC3:** 给定 fetch skeleton，When 渲染助手，Then 文字骨架按日列出；来源为 fetch 不是 make 信封。
- **AC4:** 给定 fill 循环，When 每站成功，Then 助手一行 + 主区 fetch 后累加 slot。
- **AC5:** 给定全部 key，When catalog，Then §4.6 新 key 四 locale 存在（实现时补）。
- **AC6:** 给定 mock `06-plan-skeleton.html`，When 抽检，Then 同时有 `plan-phase`、thread 叙事、多站 skeleton-day（非仅酒店）。

---

# 41 — Plan — plan-49 — 行程规划页重建（MVP-20）

**类别：** Plan · Feature **41** · 完工：**ToDo**（Story 1 Done；Story 2 实现 Done；Story 4 本切片）  
**规格：** [2play-design.md](./2play-design.md) §4.2.1 · §4.12  
**背景：** 现网 Plan 在 CTA 即 `discover`、约束条可提前填必去推荐，且骨架常只有酒店。本 Feature **重建主路径**，不在旧 `plan-page` 上继续叠叙事。

**作为** 规划用户  
**我希望** 填起飞 5 项后先由助手接手问偏好，主区只展示已填起飞限制、其余为未答占位  
**以便** 未完成 intake 时不会被搜点/必去推荐打断，也不会误以为行程已经开始生成

## 本 Feature 故事清单

| Story | 内容 | 状态 |
| --- | --- | --- |
| **1** | CTA → 助手接管；出行限制 12 格；未答（含必去）一律 `—`；无可见搜点文案 | **Done** |
| **2** | 静默建 Trip + discover（池内热度打标 ≤5）∥ b–h；g 等池；完成后 know_enough；**不** make | **实现 Done**（debug dump 由 Story 4 删除） |
| 3 | （已并入 Story 2） | — |
| **4** | intake 完成后 make + fetch 骨架（thread 展示）；**不** fill | **实现 Done**（usable 并入批次 24-P0 / 37f） |
| **5** | 步骤 b 目的地内确认起点；make 禁止无城市酒店 geocode | **Done**（2026-09-03；S7 芯片店名 2026-09-05 usable） |

## Story 1 AC

- **AC1:** 给定 Plan 起飞栏 5 项合法，When 点击「规划行程」（`play.plan.plan_cta`），Then **隐藏** `.plan-takeoff`，打开悬浮 `plan-nav`（`data-testid="plan-nav"`），助手 thread 仅问候 + 步骤 b（酒店）；**不**出现 `assistant_discovering` / `assistant_making` / `phase_making`。
- **AC2:** 给定 AC1 之后，When 主区渲染，Then 显示出行限制 panel（`data-testid="plan-constraints"`）：起飞 5 项为用户刚填的值；助手 7 项（酒店、每日开始、行程类型、节奏、交通、必去、其他）`pending`，展示 `play.plan.constraint_pending`（源英 `—`）。
- **AC3:** 给定 AC1 之后、步骤 g 尚未作答，When 渲染必去格（`data-testid="constraint-must-see"`），Then 值为 pending 占位，**不是**芯片名单、`must_see_suggestions`、或任何 POI 名。
- **AC4:** 给定 AC1 之后，When 助手 thread / 主区，Then **无** `assistant_discovering` / making / 行程列表 / tips。允许后台静默 `POST /api/plan/discover`（Story 2）。**禁止** `POST /api/plan` make。步骤 b 时芯片区不出现。
- **AC5:** 给定 AC1，When 文案，Then 全部用户可见串为 i18n key（四 locale）；测试断言 testid / key，不锁单一语言字面。

**废止（Story 1）：** 可见的 CTA discover 文案。静默 init 见 Story 2。

## Story 2 AC

- **AC6:** 给定 CTA，When 后台 init，Then 静默 `POST /api/plan/discover`：起飞 5 项写入新 Trip（内存+PG）；`discover_places` 先类目搜建池，再内部 `findIconicPlaces` **按热度从该池打** `must_see`（`max_number` 默认 5）；**不再**搜附近热点或 LLM 提名。Session cache 含 `tripId`。助手 **不** 出现 `assistant_discovering`。走 JSON dual-write，不走 NDJSON discover。
- **AC7:** 给定 b–h，When 用户每答一题，Then BFF `PATCH` 该限制入 Trip `constraints` + session；主区对应格从 pending 回填。
- **AC8:** 给定答完 f，When 任务 1 未结束，Then **不**问 g、无芯片（仅 `must_see_loading`）。进入 g 时 **必须** `fetch_trip_details`（`fields: ["candidates"]`，经 `POST /api/plan/candidates`）；芯片 = 该次 fetch 的 `must_see`（≤5）、可多选；`mustInclude` 仅用户选择。
- **AC9:** 给定 h 完成且任务 1 已结束，When 助手，Then **仅** `assistant_know_enough`（Story 4 立刻接 make）。步骤 g 仍须 `fetch_trip_details` `candidates`。**废止** Story 2 debug dump。

## Story 4 AC（骨架 only）

- **AC10:** 给定 h + discover 均完成，When 助手，Then 发送 i18n「已了解 / 正规划行程框架」（`assistant_know_enough` + `assistant_planning_skeleton`）；**禁止** `plan_next_stop` / fill / 贴士四卡；**无** debug dump。
- **AC11:** 给定 AC10，When 生成骨架，Then BFF 对**已有** `trip_id` 调 `make_itinerary`（不再 discover）；UI 进度条 + 已耗时精确到 **0.1s**（`plan-make-elapsed`）。超时或报错：i18n 友好提示，不伪造骨架。
- **AC12:** 给定 make 成功，When 助手下一条，Then i18n 标题插值目的地、天数、人数、行程类型（`assistant_skeleton_headline`）。
- **AC13:** 给定 AC12，When 再下一条，Then **只**用 `fetch_trip_details` `{ fields: ["skeleton"] }` 渲染助手 `plan-thread-skeleton`（route-spine：日/主题/站；无交通腿；无白底卡）。主区可镜像同一切片。stay-only 骨架视为失败。写信封不得当展示源。
- **AC14:** 全部新文案四 locale i18n；测试断言 key / testid。

## Story 5 AC（起点在目的地内确认 · ADR-053）

- **AC15:** 给定步骤 b 空或默认，When 提交，Then **不**调用 `search_places` / 无城市 `geocode`；不设 `dailyStart` / `originStay`；进入 c。
- **AC16:** 给定步骤 b 非空，When 提交，Then BFF 先 `POST /v1/suggest_places`（`query`=**去括号**核心名，`address`=起飞目的地；**省略** `providers[]`），对提示结果按目的地收窄（有坐标 → ≤80km；无坐标 → 名称/地址含目的地 token），住宿过滤后缺坐标则用提示**全名**再 `search_places` hydrate。提示无可用住宿卡时再 `POST /v1/search_places`（同 query/address）。命中（城市锚点 80km 内、住宿类合格）Then PATCH hotel/dailyStart 为检索名；session + `patchTrip` 写 **`originStay`**（`name`、lat、lng、`provider`、`native_id`、可解析则 `photos[0]`）；`originLat/Lng` 从卡派生；进入 c。**禁止** `geocode({ query: 仅酒店名 })`。括号副标（如「西安钟楼回民街店」）**不进**主 query。**禁止**按字种/字母长度分流（ADR-042）。
- **AC17:** 给定 b 非空且无命中，When 提交，Then **不** PATCH 起点、不进入 c；i18n `play.plan.intake_origin_not_found`；芯片 `play.plan.intake_origin_retry`（留在 b）与 `play.plan.intake_origin_skip`（按空 b 继续）。
- **AC18:** 给定 search/geocode 超时或供应商失败，When 提交非空 b，Then 不卡死；可继续且 **只传 origin.name、不传坐标 / 无 originStay 指针**。
- **AC19:** 给定 Hills Hotel Lisboa + 目的地里斯本，When 目的地内搜索，Then **不得**因澳门同名点判未命中。
- **AC20:** 给定 make，When 组装 origin，Then 使用 Story 5 已解析的 `originStay`（或坐标），否则仅 name；**删除** skeleton 路径上对 `dailyStart` 的无城市 geocode。
- **AC21:** 给定 skip（空 b / 忽略起点），When 提交，Then 可有城市坐标；**无**酒店店卡；图空；禁止用城市点冒充某家酒店。

---

# 8 字段起飞 + needs_input 问卷 — `2play-plan-90a`

**类别：** 2play · 状态：**Done**（2026-09-09）  
**ADR：** ADR-050（无产品 LLM）、ADR-052（省略 providers[]）  
**依赖：** `agent-itinerary-93a`

**作为** 已登录出行者  
**我希望** 填 8 项结构化表单后逐题回答 agent 的 4 问  
**以便** 不写死问题顺序、不在 2play 调产品 LLM

### US1 — 8 项提交调 plan_trip

**AC1**

Given 起飞条 8 项（city, startDate, days, party, budget, tripType, pace, transit）  
When 提交  
Then BFF `POST /api/plan/trip` Zod 校验后调 agent `plan_trip`（**不传 providers[]**）  
And 成功 200 时写入 `planSessionCache`（8 项 + `tripId` / `revision`）  
And 渲染 `need_input.questions`  
And 用户可见串为 i18n key  
And 调试页在 Q1 前可通过 `GET /api/plan/current` + `stops-pool?city=` 读到 trip 与城市 registry 池

### US2 — 逐题渲染，一次提交

**AC2**

Given agent 一次返回 4 题  
When 助手  
Then 一题一题输入  
And 不逐题重调 `plan_trip`  
And 四题均 PATCH `/api/plan/session`（12 输入与 debug 同步）  
And 答完收口 `fetch_trip_details` candidates（T1 不带 origin 二次 `plan_trip`，以免全环）  
And hotel 空提交 = 跳过验真（PATCH skip）；非空验真失败停留本题（`play.plan.intake_origin_not_found`，不吞 422）  
And must_see 芯片多选

### US3 — 无产品 LLM；as-built 暂停

**AC3**

Given T1 路径  
When 规划  
Then 2play 不调用产品 LLM  
And as-built discover→make→plan_next_stop 标 Paused（不删代码）

### US4 — 最小 candidates；T1 不 make/fill

**AC4**

Given 4 问已提交  
When 展示  
Then `fetch_trip_details` candidates 最小渲染  
And T1 **不**调用 `POST /api/plan` NDJSON / `runPlan` 骨架或 fill（T2）  
And 助手收口为 `play.plan.assistant_know_enough`

---

# 起点满匹配才自动命中 — `2play-plan-96`

**类别：** 2play · 状态：**Done**（2026-09-09；AC5 suggest 优先同日补）  
**依赖：** `2play-plan-90a`

**作为** 出行者  
**我希望** 只有整名对上且唯一时才不等确认  
**以便** 「三台」不会悄悄变成路名分店

### AC1 — 满匹配唯一

Given 查询与唯一候选店名整词/整句匹配（例 Hills Hotel Lisbon）  
When 提交住宿  
Then 自动采用该店，不停留候选列表

### AC2 — 部分匹配唯一

Given 查询仅为部分字（例「三台」落在「三台山路…分店」）且搜到至少一家住宿  
When 提交  
Then 不自动 hit  
And 线程只显示 `play.plan.intake_origin_candidates`（插值目的地 + 用户输入），**不**显示 `need_prompt.hotel`  
And 其下纵向列出全部候选  
And 可点候选 / 重输发送 / 跳过 / 空发送  
And 再次提交非候选文本时先清上一轮候选再搜

### AC3 — 多候选

Given 多家住宿  
When 提交  
Then 同 AC2，不 auto-hit

### AC2b — 零匹配

Given 目的地附近 0 张合格住宿卡  
When 提交非空住宿  
Then 线程只显示 `play.plan.intake_origin_not_found`  
And **不**显示 `need_prompt.hotel`、无候选芯片  
And 可重输 / 跳过 / 空发送 / 重答上一题  
And 再次提交时清掉上一轮「找不到」后再搜

### AC4 — 满匹配定义

Given 归一化 query 是 name 连续子串  
When 该子串仅出现在路/街/巷/号地址片段，或 CJK query 长度小于 4 且不等于店名主号  
Then 视为部分匹配（「三台」部分；「三台山庄」对「三台山庄」满匹配）

### AC5 — 补全优先（suggest → search）

Given 用户输入短前缀（例 `SFEE`）且目的地杭州  
When 提交住宿  
Then 先走 `suggest_places`（高德 inputtips / Google autocomplete，限城）  
And 能得到目标店（例 SFEEL…）时**不得**因 `search_places`  alone 空结果判 `not_found`  
And 外城同品牌提示须被目的地过滤丢掉  
And 提示为空时仍回退 `search_places`（例完整英文店名）

---

# 重答上一题 / 跳过这一题 — `2play-plan-97`

**类别：** 2play · 状态：**Done**（2026-09-09）  
**依赖：** `2play-plan-90a`

**作为** 出行者  
**我希望** 问卷里能跳过本题或改上一题  
**以便** 不用靠「空发送」或整段重来

### AC1 — 跳过可见且生效

Given 当前一题未完成  
When 选择「跳过这一题」  
Then 本题按空答提交（酒店 = 验真 skip；时间默认 09:00；必去/其他空）  
And 进入下一题或四问收口  
And 用户泡泡为已跳过文案（i18n）

### AC2 — 重答上一题

Given 至少已答完一题  
When 选择「重答上一题」  
Then 回到上一题，该题与本题草稿清空  
And 不重跑 `plan_trip`、不丢 trip_id  
And 第一题时「重答」不可用或无效果

### AC3 — 文案

Given 任意 locale（CN/EN/HK/TW）  
When 渲染题面与页脚  
Then 不再出现「点发送即可跳过」  
And 通顺指向「跳过」  
And 主发送键仍表示提交有内容的答案  
And 题面用 i18n / 覆盖模型英文里的 send 句

### AC4 — 两路径

Given agent 四问或本地 b–h  
When 重答/跳过  
Then 行为一致

### AC5 — 布局

Given 必去多选或住宿验证候选  
When 渲染过程芯片  
Then 芯片在对话线程（当前助手气泡下），`plan-nav__quick--stack` 一行一点  
And dock 不含过程芯片  

Given 问答进行中（含可重答窗口）  
When 渲染助手面板  
Then 「跳过这一题」「重答上一题」固定在输入框上方左侧（`plan-nav__need-actions`），不进芯片行、不随线程滚走

---

# stop 标源 — `2play-plan-98`

**类别：** 2play · 状态：**Done**（2026-09-09）

**作为** 调试者与出行者  
**我希望** 每个停靠能看出 Google 或高德  
**以便** 判断池与行程来源

### AC1 — Debug

Given 有 registry / trip candidates / 骨架或行程停靠  
When 打开 debug  
Then 各表有源列：Google / 高德 / —

### AC2 — 行程主区

Given 起点、景点或餐厅 slot 带 `provider`  
When 渲染行程  
Then 可见对应源标（i18n）  
And 无 provider 显示 —

### AC3 — 数据

Given agent 卡已有 `provider`  
When BFF 组 pool / stops-pool / itinerary  
Then 不丢 `provider`，不编造

---

# 稳定 key + 不重提名 — `2play-plan-99`

**类别：** 2play · 状态：**Done**（2026-09-09）  
**依赖：** `2play-plan-90a`、`agent-itinerary-95` AC5  
**ADR：** ADR-042

**作为** 出行者  
**我希望** 起飞字段按目录 key 传给 agent，且换界面语言不重跑必去提名  
**以便** 约束条与 L3 文案对得上，且不重复烧提名

### AC1 — 稳定 key

Given 起飞条 `tripType` / `budget` / `pace` / `transit`  
When BFF 组 `plan_trip`  
Then 传 `trip_type` / `budget` / `pace` / `transit_preference` 为稳定 key（如 `couple`、`comfort`|`mid`|`luxury`|`economy`、`medium`、`transit_walk`）  
And **不**把 `comfort` 塌成 `premium`  
And **不**传 `providers[]`

### AC2 — 不重提名

Given 同一 `trip_id` 已完成提名  
When 仅切换 UI locale 再读行程 / 再调 `plan_trip`  
Then 不第二次调用 `nominate_must_see`（agent `agent-itinerary-95` AC5）  
And 芯片 POI 集合不变

---

# Takeoff 11 → submit — `2play-plan-100`

**类别：** 2play · MVP-T2 · 状态：**Implemented**（ADR-064 confirm + dest label patch pending usable confirm 2026-09-10；原 usable 2026-09-09）  
**ADR：** [ADR-061](../adr/ADR-061-takeoff-11-fields-skeleton-first.md)（Accepted for T2）· [ADR-064](../adr/ADR-064-takeoff-submit-confirm-origin-first.md)（Option A）  
**Mock SoT：** [`ui-mockup/06-plan-takeoff-11.html`](./ui-mockup/06-plan-takeoff-11.html)  
**依赖：** agent geocode 结构化返回（`agent-geocode-100`）  
**非目标：** 提交后助手接管、`plan_trip` 骨架先行、去掉固定 4 问、必去提名（属 MVP-T3 / `2play-plan-101`）

**作为** 已登录出行者  
**我希望** 在起飞栏一次填齐 11 项边界并完成 blur 验真后再提交  
**以便** 不必在助手里再答住宿 / 开始时间 / 其他；必去点不在起飞栏

### US1 — 11 字段布局与控件

**AC1**

Given Plan 页起飞栏  
When 渲染  
Then 展示 11 字段：destination · startDate · tripType · days · partySize · budget · pace · origin · startTime · transit · other  
And 布局为两行共享七列轨，对齐 mock `06-plan-takeoff-11.html`（`data-testid="plan-takeoff-11"` / `plan-takeoff-row-1` / `plan-takeoff-row-2`）  
And **无** must-see / 必去地输入  
And tripType = 下拉+手动（combo）；budget / pace / transit = 纯下拉；startDate = 日期；days / partySize = `type=number` 原生 spinner（**无** ± 按钮）；startTime = 时间（标签无必填星号）；origin / other / destination = 文本  
And CN 标签与 mock 一致：目的地 · 行程开始日期 · 行程类型 · 天数 · 人数 · 行程预算 · 动线节奏 · 行程起点 · 每日出发时间 · 交通偏好 · 其他要求  
And 全部用户可见串为 i18n key（EN/CN/HK/TW）

### US2 — 目的地 blur 验真标签

**AC2**

Given 用户输入目的地后焦点离开  
When BFF 调 agent `geocode`（省略 `providers[]`）成功  
Then 显示结构化标签：国内/台港例 `中国台湾/台北`、`中国/杭州`；海外例 `葡萄牙/里斯本(Lisbon)`（有 `city_en` 且与 `city` 不同时括号补英文）  
And 输入框文案不改写（仍为用户输入，如 `里斯本` / `杭州`）  
And `data-testid="plan-dest-verified"`  
And geocode 失败：不编造标签；字段 invalid + i18n 错误；提交保持禁用

### US3 — 行程起点 blur 悬浮窗

**AC3**

Given 用户输入 origin 后焦点离开  
When BFF 走现有 `suggest_places` 验真（目的地内收窄，同 `2play-plan-96` 语义）  
Then  
- 唯一满匹配 → 静默 hit，不弹层  
- 部分匹配 → **页面悬浮窗**列出候选（`plan-origin-overlay` / `plan-origin-candidates`），可选 / 重输 / 跳过  
- 不匹配 → 悬浮窗 not_found（`plan-origin-not-found`）+ 重输 / 跳过  
And 跳过允许不设起点继续提交

### US4 — 提交门禁与 body

**AC4**

Given 必填字段未通过验真（destination 无 verified、days/party 非法等）  
When 看 CTA  
Then `plan-submit` disabled  

Given 11 项合法（origin/other/startTime 可空，空 startTime 默认按每日 09:00 语义）  
When 用户完成提交确认（US6）  
Then BFF 接受并映射 `origin` / `startTime` / `other` 进 agent body 字段（与既有 `plan_trip` 调用并存）  
And **不**改变 agent 固定 4 问 intake 环（T3）  
And 用户可见错误为 i18n key

### US5 — startTime 语义

**AC5**

Given `startTime` 有值或默认 09:00  
When 写入边界  
Then 表示**每日**行程开始时间（非仅 Day-1）

### US6 — 提交确认弹层（ADR-064 · Option A）

**AC6**

Given 起飞栏必填验真已通过  
When 用户在任意起飞字段按 Enter，或点击「规划行程」  
Then **不**立即调用 `plan_trip`  
And 打开提交确认弹层（`plan-submit-confirm-overlay`）：标题/正文/摘要 `dl`（Option A）/「返回修改」/「确认规划」  
And 弹层视觉与起点悬浮窗同壳  

Given 起点非空且 resolve 为部分匹配或不匹配  
When Enter /「规划行程」  
Then **仅**打开起点悬浮窗；确认弹层保持关闭  

Given 起点悬浮窗中用户选择候选或跳过  
When 关闭起点层  
Then **再**打开提交确认弹层  

Given 起点悬浮窗中用户「重新输入」  
When 关闭  
Then 不打开确认弹层；焦点回到起点  

Given 用户在确认弹层点「确认规划」  
When 确认  
Then 才进入既有提交 / T3 助手接管路径  

Given 用户点「返回修改」或点遮罩  
When 关闭  
Then 留在起飞栏；不提交
---

# Submit → assistant + skeleton — `2play-plan-101`

**类别：** 2play · MVP-T3 · 状态：**Done**（usable Confirmed 2026-09-11）  
**ADR：** [ADR-062](../adr/ADR-062-mvp-t3-skeleton-vs-t4-nominate.md)（切片边界）· [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)（发现路径 Target）· [ADR-061](../adr/ADR-061-takeoff-11-fields-skeleton-first.md)（T2 Done）· [ADR-050](../adr/ADR-050-where2play-no-product-llm.md)  
**依赖：** `2play-plan-100` usable；agent `agent-itinerary-100`  
**配对：** `agent-itinerary-100`  
**Mock / UI：** [`ui-mockup/06-plan-assistant-t3.html`](./ui-mockup/06-plan-assistant-t3.html)（接管 + 进度 + 框架）；起飞入口 [`06-plan-takeoff-11.html`](./ui-mockup/06-plan-takeoff-11.html)；观测 [`/debug/plan`](../../where2play/app/(app)/debug/plan/page.tsx)  
**非目标：** 固定四问 intake；必去提名/聊天 refine（→ `2play-plan-102`）；`plan_next_stop` fill / meals / directions / 贴士全量写路径

**作为** 已登录出行者  
**我希望** 提交起飞栏后由助手接管、创建行程并看到行程框架与后端进度  
**以便** 在聊天 refine / 必去提名之前先确认行程已成立且框架可读

### US0 — Specs / 临时文件清理（DoD 门禁）

**AC0**（规则向）

- [x] `plan.md` / backlog / ADR 链接不指向已废聊天 dump 作为 SoT
- [x] `mvp-1t-closing-plan.md` 已归档或删除（收尾已 Confirmed）
- [x] `agent-plus-2play-chat.md` / `real-agent-refactory-chat.md` 已移至 `specs/archive/` 或删除，并在 `change-log` 记一笔
- [x] `agent-specs/tmp-0909.md` 耐久内容已并入 ADR-061/062 / `agent-design` 后 stub 或删除
- [x] `real-agent-refactory.md` / `draft-nominate-must-see-prompt.md` 仍为短指针或已改链到 durable 真源

### US1 — 助手接管且无四问

**AC1**

```gherkin
Scenario: 提交起飞后助手接管且不进入固定四问
  Given 已登录用户在 Plan 页完成起飞 11 项合法输入
  And 目的地验真已通过
  When 用户提交「规划行程」
  Then 起飞栏不再作为主编辑区展示
  And 行程助手面板打开
  And 助手不提出住宿/每日出发时间/必去点/其他要求这四道固定问卷题
  And 主区展示只读出行限制（起飞边界已填；每日起点/出发时间/其他来自起飞栏）
```

### US2 — 创建行程并得到 trip id

**AC2**

```gherkin
Scenario: 提交后创建持久行程
  Given 起飞 11 项合法且可提交
  When 用户提交「规划行程」
  Then 系统向行程编排服务请求创建行程
  And 用户会话持有一个非空行程标识
  And 调试页可读取到同一行程标识与起飞边界
```

**AC2b**

```gherkin
Scenario: 创建行程失败时诚实报错
  Given 起飞 11 项合法
  And 行程编排服务不可用或返回失败
  When 用户提交「规划行程」
  Then 助手或主区展示可理解的错误（i18n key）
  And 不假装已生成行程框架
  And 不泄露服务端密钥或堆栈
```

### US3 — 助手进度文案

**AC3**

```gherkin
Scenario: 助手用进度句说明后端阶段
  Given 用户已提交起飞并开始创建行程
  When 编排服务报告阶段进展（例如已建行程、正在生成框架、框架就绪）
  Then 助手对话区按顺序出现对应进度文案
  And 文案来自产品 i18n 目录（非 where2play 本地大模型旁白）
  And 进度句不要求用户回答固定四问
```

### US4 — 主区展示行程框架

**AC4**

```gherkin
Scenario: 框架就绪后主区用现行 UI 展示
  Given 行程已创建且内部 skeleton 已写入
  When 客户端拉取行程详情中的框架（fetch skeleton）
  Then 主区按现行 as-built 结构展示日程与框架停点
  And 展示出行限制只读面板
  And 不展示已填充的逐站详情交通腿或餐厅填充分（T3 不做 fill）
```

**AC4b**

```gherkin
Scenario: 框架拉取失败
  Given 行程标识已存在但框架读取失败
  When 主区尝试展示行程
  Then 用户看到可恢复的错误或重试引导（i18n）
  And 助手进度不声称「行程框架已经规划完毕」
```

### US5 — 调试页：行程与 stops-pool

**AC5**

```gherkin
Scenario: 调试页展示当前行程与城市景点池
  Given 用户已提交起飞并获得行程标识
  When 用户打开行程调试页
  Then 页面展示当前行程标识与起飞边界摘要
  And 页面展示目的地城市的 stops-pool（景点池）条目或明确空态
  And 不要求新做独立调试产品；复用现有调试页能力
```

### 交叉约束

- 全部用户可见字符串为 i18n key（EN/CN/HK/TW）。
- **用户可见用语「框架」；禁止 UI 使用「骨架」**（内部仍称 skeleton / 骨架）— 见 `2play-design` §4.7.1。
- 助手须读说明句用 `.bubble--agent-notice`（白底聊天气泡，与用户 peach 气泡对称）。
- where2play 不调用产品 LLM 生成进度散文（ADR-050）。
- 必去提名、聊天改框架属 `2play-plan-102` / MVP-T4。

---

# Assistant deviations text — `2play-plan-103`

**类别：** 2play · MVP-T3++ · 状态：**AC Ready**  
**ADR：** [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md) todo5 UI  
**依赖：** `agent-discover-110c`；`2play-plan-101` Done  
**配对：** `agent-discover-110c`  
**非目标：** 新警告面板/模态；聊天 refine（→ T4）

**作为** 已登录出行者  
**我希望** 在助手窗口骨架下方看到不符合项与原因的文字说明  
**以便** 自行评估是否接受当前框架

### AC

```gherkin
Scenario: Deviations render as text under skeleton
  Given trip skeleton fetch includes deviations[]
  When Plan assistant shows the framework
  Then below the day framework, text lists each non-conformance + reason
  And no separate warning panel / modal is required
  And all user-visible strings are i18n keys (EN/CN/HK/TW)
  And empty deviations → no extra block
```

---

# Expand-radius confirm UI — `2play-plan-104`

**类别：** 2play · MVP-T3++ · 状态：**AC Ready**  
**ADR：** [ADR-067](../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md) todo6b  
**依赖：** `agent-discover-110d`；`2play-plan-103`  
**配对：** `agent-discover-110d`  
**非目标：** 自动混入周边城市景点

**作为** 已登录出行者  
**我希望** 在 POI 不足时确认或拒绝扩大搜索半径  
**以便** 控制是否纳入周边景点

### AC

```gherkin
Scenario: User confirms or declines expand radius
  Given agent returns need_input for expand-radius proposal
  When assistant shows the confirm/decline affordance
  Then affirm posts answer and planning continues with expanded grounding
  And decline continues with local-only + any deviation text
  And copy is i18n keys; no product LLM prose
```
