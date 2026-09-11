# Prompt 探针测试用例

5 组真实起飞 11 项输入，用于发现步骤提示词 A/B/优化A 对比探针。
字段对齐 places-agent 真实 catalog key（`places-ontology.ts` TRIP_TYPE / BUDGET / PACE / TRANSIT）。

## 字段说明

| 字段 | catalog key 来源 |
| --- | --- |
| trip_type | TRIP_TYPE：family_kids / couple_romance / city / solo / food_checkin / family_vacation / friends / business |
| budget | BUDGET：economy / mid / comfort / luxury |
| pace | PACE：tight / medium / relaxed |
| transit | TRANSIT：transit_walk（公交地铁优先）/ drive_walk（打车优先） |

当用户意图无直接对应 trip_type（如"探访历史""动漫主题"），trip_type 取最接近的可选项，具体意图写入 `other`。

---

## test 1 — 上海亲子

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

## test 2 — 杭州情侣

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

## test 3 — 西安历史

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

## test 4 — 里斯本情侣

- 城市：里斯本
- 日期：2026-09-16
- trip_type：couple_romance（情侣浪漫）
- 天数：3
- 人数：2
- budget：luxury（豪华）
- pace：medium（适中）
- transit：transit_walk（公交+步行）
- 起点：Hills Hotel Lisboa
- 出发时间：07:00
- other：（无）

## test 5 — 东京动漫

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
