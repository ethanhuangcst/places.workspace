# E2E-11 布拉格（Prague）2天行程

> 本文件由 `scripts/e2e-places-agent.py` 自动生成，模拟用户调用 places-agent 工具链路得到的真实结果。

## 模拟用户输入（8 行表单）

| 字段 | 值 |
| --- | --- |
| 城市 | 布拉格（Prague） |
| 出发日期 | 2026-10-10 |
| 天数 | 2 |
| 酒店 | （未提供） |
| 节奏 | relaxed |
| 预算 | 1（节约） |
| 兴趣 | 老城、啤酒 |
| 必去 | （用户未选择，走目的地无关路径） |

## places-agent 工具链路

1. `geocode`（有酒店时）→ `travel_tips`（记录 `iconic_places`，ADR-045 展示源）→ 2. `discover_places` → 3. `make_itinerary` → 4. `plan_next_stop` 链直到 `trip_complete`（F65：无 `display_current_stop`）

## 工具调用记录

| # | 工具 | 结果 | 耗时(s) |
| --- | --- | --- | --- |
| 1 | geocode | ✓ skipped(no hotel) |  |
| 2 | travel_tips | ✓ iconic= | 4.53 |
| 3 | discover_places | ✓ places=40, restaurants=32 | 9.5 |
| 4 | make_itinerary | ✓ next=plan_next_stop | 14.92 |
| 5 | plan_next_stop | ✓ next=plan_next_stop | 1.17 |
| 6 | plan_next_stop | ✓ next=plan_next_stop | 4.54 |
| 7 | plan_next_stop | ✓ next=plan_next_stop | 7.87 |
| 8 | plan_next_stop | ✓ next=plan_next_stop | 5.38 |
| 9 | plan_next_stop | ✓ next=plan_next_stop | 5.73 |
| 10 | plan_next_stop | ✓ next=plan_next_stop | 17.75 |
| 11 | plan_next_stop | ✓ next=plan_next_stop | 0.41 |
| 12 | plan_next_stop | ✓ next=plan_next_stop | 2.94 |
| 13 | plan_next_stop | ✓ next=plan_next_stop | 4.32 |
| 14 | plan_next_stop | ✓ next=plan_next_stop | 21.64 |
| 15 | plan_next_stop | ✓ next=plan_next_stop | 7.31 |
| 16 | plan_next_stop | ✓ next=plan_next_stop | 3.88 |
| 17 | plan_next_stop | ✓ next=trip_complete | 9.23 |

## 结果：成功（trip_complete）

**Trip Store:** `trip_id=cmudp3q8f006802rgkx6xos6e` · `revision=16`

## 骨架

- **Day 1** 老城与查理大桥经典线：查理大桥 → 老城桥塔 → lunch → 布拉格天文钟 → 火药塔 → dinner
- **Day 2** 布拉格城堡历史高地：布拉格城堡 → 圣维特主教座堂 → 黄金巷 → lunch → 小城桥塔 → 维巴庭园 → dinner

## 逐站填充结果

### 查理大桥  · attraction
- 时段：09:00 – 09:45

### 老城桥塔  · attraction
- 时段：09:51 – 10:36
- 到达：walk 约 2 分钟
- 备注：station_timing_adjusted

### Restaurant Mlýnec  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 2 分钟
- 评分：4.7
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=50.085420899999995%2C14.413601)
- [google_app](https://maps.google.com/?q=50.085420899999995%2C14.413601)
- [amap_web](https://uri.amap.com/marker?position=14.413601,50.085420899999995&name=Restaurant%20Ml%C3%BDnec)

### 布拉格天文钟  · attraction
- 时段：12:55 – 13:40
- 到达：walk 约 25 分钟
- 备注：station_timing_adjusted

### 火药塔  · attraction
- 时段：13:50 – 14:35
- 到达：walk 约 7 分钟
- 备注：station_timing_adjusted

### TastyTayfun  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 3 分钟
- 评分：5
- 类别：dessert_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=50.086241%2C14.4278789)
- [google_app](https://maps.google.com/?q=50.086241%2C14.4278789)
- [amap_web](https://uri.amap.com/marker?position=14.4278789,50.086241&name=TastyTayfun)
- 备注：station_timing_adjusted

### 布拉格城堡  · attraction
- 时段：09:00 – 09:45

### 圣维特主教座堂  · attraction
- 时段：09:46 – 10:31
- 到达：walk 约 1 分钟

### 黄金巷  · attraction
- 时段：10:38 – 11:23
- 到达：walk 约 4 分钟
- 备注：station_timing_adjusted

### Lore Malastrana  · meal
- 时段：11:37 – 12:37
- 到达：walk 约 14 分钟
- 评分：4.8
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=50.089174%2C14.402206500000002)
- [google_app](https://maps.google.com/?q=50.089174%2C14.402206500000002)
- [amap_web](https://uri.amap.com/marker?position=14.402206500000002,50.089174&name=Lore%20Malastrana)
- 备注：station_timing_adjusted

### 小城桥塔  · attraction
- 时段：13:16 – 14:01
- 到达：walk 约 39 分钟
- 备注：station_timing_adjusted

### 维巴庭园  · attraction
- 时段：14:09 – 14:54
- 到达：walk 约 5 分钟
- 备注：station_timing_adjusted

### Yellow #zestanku  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 1 分钟
- 评分：4.9
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=50.0873553%2C14.403950700000001)
- [google_app](https://maps.google.com/?q=50.0873553%2C14.403950700000001)
- [amap_web](https://uri.amap.com/marker?position=14.403950700000001,50.0873553&name=Yellow%20%23zestanku)
