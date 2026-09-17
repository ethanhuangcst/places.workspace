# Prompt 探针测试用例

12 组真实起飞 11 项输入，用于发现步骤提示词 A/B/优化A 对比探针，并覆盖供应商路由、POI 密度边界、locale、跨语言名匹配等 case。
字段对齐 places-agent 真实 catalog key（`places-ontology.ts` TRIP_TYPE / BUDGET / PACE / TRANSIT）。

## 设计原则

- **成本优先**：7 个 AMAP 用例（免费）+ 5 个 Google 用例（收费，走 probe 缓存 `PLACES_PROBE_CACHE_DIR`，`GOOGLE_DAILY_BUDGET_CALLS` 限流）。
- **维度覆盖**：8 种 trip_type、4 种 budget、3 种 pace、2 种 transit、2–5 天、CN/EN locale、有/无起点。
- **边界 case**：expand-radius（POI 不足）、无起点、跨语言名匹配、多日远集群、HK 双供应商路由、TW 输出措辞。
- **默认 CI**：7 AMAP case 走 fixture；Google case 为 opt-in（`make test-e2e-caller`）。

## 字段说明

| 字段 | catalog key 来源 |
| --- | --- |
| trip_type | TRIP_TYPE：family_kids / couple_romance / city / solo / food_checkin / family_vacation / friends / business |
| budget | BUDGET：economy / mid / comfort / luxury |
| pace | PACE：tight / medium / relaxed |
| transit | TRANSIT：transit_walk（公交地铁优先）/ drive_walk（打车优先） |

当用户意图无直接对应 trip_type（如"探访历史""动漫主题"），trip_type 取最接近的可选项，具体意图写入 `other`。

## 维度覆盖矩阵

| 维度 | 覆盖值 | 对应用例 |
| --- | --- | --- |
| 供应商 | AMAP-only | 1–3, 6–9 |
| | Google-only | 4–5, 11–12 |
| | 双路由(HK) | 10 |
| trip_type | family_kids | 1, 10 |
| | couple_romance | 2, 4 |
| | city | 3, 12 |
| | solo | 5 |
| | food_checkin | 6 |
| | family_vacation | 7 |
| | friends | 8, 11 |
| | business | 9 |
| budget | economy | 8, 11 |
| | mid | 1, 3, 6, 5, 10 |
| | comfort | 7, 9, 12 |
| | luxury | 2, 4 |
| pace | tight | 3, 5, 9 |
| | medium | 1, 6, 4, 10, 12 |
| | relaxed | 2, 7, 11 |
| 天数 | 2 | 6, 9 |
| | 3 | 1–3, 8, 5, 10, 12 |
| | 4 | 4, 11 |
| | 5 | 7 |
| 起点 | 有 | 1, 2, 6, 7, 9, 4, 5, 10, 12 |
| | 无 | 3, 8, 11 |
| locale | CN | 1–10, 12 |
| | EN | 11 |
| 边界 case | expand-radius | 4, 11 |
| | 跨语言名匹配 | 4 (PT↔EN) |
| | HK 双路由 | 10 |
| | TW 输出措辞 | 12 |
| | 无起点 | 3, 8, 11 |
| | 多日远集群 | 7 (5d) |

## 断言

通用断言（所有 12 case）：
- `status === "ready"`（非 `failed`、非 `needs_input`）
- `skeleton.days.length === numDays`
- 每天至少 1 个 attraction stop
- 无捏造 POI（pool 卡片有 `sources[].native_id`）
- `deviations`（如有）显示为文本，非硬失败

特定断言：
- **case 4 (里斯本 4d)**：触发 expand-radius `need_input` 或 deviation；确认扩大后 `status === "ready"`；Belém/Jerónimos 有图
- **case 10 (香港)**：pool 含 AMAP + GOOGLE 双源卡片
- **case 11 (曼谷 4d EN)**：`need_input` expand-radius 或 deviation；EN locale 输出
- **case 12 (台北)**：TW 措辞（非 HK 措辞）
- **case 3 (西安无起点)**：geocode 城市为 anchor，骨架正常
- **case 7 (北京 5d)**：5 天骨架无远集群漂移

## 执行策略

| 层 | 机制 | 成本 |
| --- | --- | --- |
| 默认 CI | 7 AMAP case 走 fixture；Google case 跳过 | 零供应商 $ |
| opt-in probe | `make test-e2e-caller` 跑全部 12 case | L1 缓存复用 |
| Google 配额 | `GOOGLE_DAILY_BUDGET_CALLS=50` | 硬日上限 |
| 缓存 | `PLACES_PROBE_CACHE_DIR=tmp/.probe-cache` | 热缓存后 ≈ 免费 |

### 探针已知问题修复（2026-09-17）

| 问题 | 修复 |
| --- | --- |
| fill 挂起、日 tab 排队 | where2play 300s abort + abrupt-close → `assistant_fill_timeout` |
| `skeleton_patched` 无限重试 | 同 stop 最多 3 次后强制前进 |
| test10 香港酒店名当景点 | prompt 排除 + `dropUnknownAttractionStops` 丢 stay 同名 attraction |
| test6 成都 nominate 偶发 pool=1 | nominate &lt;8 名时重试一次 |
| test12 台北 Google pool 薄 | discover `bias_radius_m=50_000`（目的地无关） |

---

## AMAP 用例（免费）

### test 1 — 上海亲子

- 城市：上海
- 日期：2026-09-16
- trip_type：family_kids（亲子玩乐）
- 天数：3
- 人数：3
- budget：mid（适中）
- pace：medium（适中）
- transit：drive_walk（打车+步行）
- 起点：上海虹桥中心爱琴海亚朵S酒店
- 出发时间：09:00
- other：7岁男孩

### test 2 — 杭州情侣

- 城市：杭州
- 日期：2026-09-16
- trip_type：couple_romance（情侣浪漫）
- 天数：3
- 人数：2
- budget：luxury（豪华）
- pace：relaxed（轻松）
- transit：drive_walk（打车+步行）
- 起点：SFEEL设计师酒店(杭州西湖武林广场店)
- 出发时间：09:00
- other：（无）

### test 3 — 西安历史

- 城市：西安
- 日期：2026-09-16
- trip_type：city（城市漫游 — 系统无 history key，取最接近项）
- 天数：3
- 人数：3
- budget：mid（适中）
- pace：tight（紧凑）
- transit：transit_walk（公交+步行）
- 起点：（无指定起点）
- 出发时间：09:00
- other：探访历史

### test 6 — 成都美食探店

- 城市：成都
- 日期：2026-09-16
- trip_type：food_checkin（美食打卡）
- 天数：2
- 人数：2
- budget：mid（适中）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：成都博舍
- 出发时间：10:00
- other：川菜和小吃探店

### test 7 — 北京家庭度假

- 城市：北京
- 日期：2026-09-16
- trip_type：family_vacation（家庭度假）
- 天数：5
- 人数：4
- budget：comfort（舒适）
- pace：relaxed（轻松）
- transit：drive_walk（打车+步行）
- 起点：北京王府井文华东方酒店
- 出发时间：09:00
- other：含老人，节奏慢

### test 8 — 厦门朋友出行

- 城市：厦门
- 日期：2026-09-16
- trip_type：friends（朋友出行）
- 天数：3
- 人数：4
- budget：economy（经济）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：（无指定起点）
- 出发时间：09:30
- other：海边和文艺景点

### test 9 — 深圳商务

- 城市：深圳
- 日期：2026-09-16
- trip_type：business（商务出行）
- 天数：2
- 人数：1
- budget：comfort（舒适）
- pace：tight（紧凑）
- transit：drive_walk（打车+步行）
- 起点：深圳柏悦酒店
- 出发时间：08:00
- other：白天会议，晚上自由

---

## Google 用例（收费，opt-in + 缓存）

### test 4 — 里斯本情侣 4 日

- 城市：里斯本
- 日期：2026-09-16
- trip_type：couple_romance（情侣浪漫）
- 天数：4
- 人数：2
- budget：luxury（豪华）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：Hills Hotel Lisboa
- 出发时间：07:00
- other：（无）
- 边界 case：4 天 POI 密度不足触发 expand-radius；骨架用葡语名（Torre de Belém），pool 用英语名（Belém Tower），验证跨语言名匹配与图片解析

### test 5 — 东京动漫

- 城市：东京
- 日期：2026-09-16
- trip_type：solo（个人放松 — 系统无 anime key，1人独行取 solo）
- 天数：3
- 人数：1
- budget：mid（适中）
- pace：tight（紧凑）
- transit：transit_walk（公交+步行）
- 起点：Hotel Monterey Lasoeur Ginza
- 出发时间：08:00
- other：80年代动漫粉丝，动漫主题度假

### test 10 — 香港亲子

- 城市：香港
- 日期：2026-09-16
- trip_type：family_kids（亲子玩乐）
- 天数：3
- 人数：3
- budget：mid（适中）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：香港中环文华东方酒店
- 出发时间：09:00
- other：8岁女孩
- 边界 case：HK 双供应商路由（AMAP + GOOGLE），验证 pool 含双源卡片；酒店名不含景点词，避免 LLM 把 lodging 当 attraction

### test 11 — 曼谷朋友 4 日

- 城市：曼谷
- 日期：2026-09-16
- trip_type：friends（朋友出行）
- 天数：4
- 人数：3
- budget：economy（经济）
- pace：relaxed（轻松）
- transit：transit_walk（公交+步行）
- 起点：（无指定起点）
- 出发时间：10:00
- other：夜市和寺庙
- 边界 case：4 天 POI 密度边界（expand-radius）；EN locale；无起点

### test 12 — 台北城市漫游

- 城市：台北
- 日期：2026-09-16
- trip_type：city（城市漫游）
- 天数：3
- 人数：2
- budget：comfort（舒适）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：台北晶华酒店
- 出发时间：09:00
- other：（无）
- 边界 case：TW 输出措辞（非 HK 措辞），验证 locale 回退顺序
