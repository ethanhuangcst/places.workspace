# E2E-01 里斯本（Lisbon）4天行程

> 本文件由 `scripts/e2e-places-agent.py` 自动生成，模拟用户调用 places-agent 工具链路得到的真实结果。

## 模拟用户输入（8 行表单）

| 字段 | 值 |
| --- | --- |
| 城市 | 里斯本（Lisbon） |
| 出发日期 | 2026-10-10 |
| 天数 | 4 |
| 酒店 | Hills Hotel Lisboa |
| 节奏 | relaxed |
| 预算 | 3（宽松） |
| 兴趣 | 历史建筑、海边风景、美食 |
| 必去 | 贝伦区、辛特拉、卡斯凯什 |

## places-agent 工具链路

1. `geocode`（有酒店时）→ `travel_tips`（记录 `iconic_places`，ADR-045 展示源）→ 2. `discover_places` → 3. `make_itinerary` → 4. `plan_next_stop` 链直到 `trip_complete`（F65：无 `display_current_stop`）

## 工具调用记录

| # | 工具 | 结果 | 耗时(s) |
| --- | --- | --- | --- |
| 1 | geocode | ✓  | 0.96 |
| 2 | travel_tips | ✓ iconic= | 5.36 |
| 3 | discover_places | ✓ places=49, restaurants=40 | 11.84 |
| 4 | make_itinerary | ✓ next=plan_next_stop | 30.27 |
| 5 | plan_next_stop | ✓ next=plan_next_stop | 5.5 |
| 6 | plan_next_stop | ✓ next=plan_next_stop | 4.9 |
| 7 | plan_next_stop | ✓ next=plan_next_stop | 7.3 |
| 8 | plan_next_stop | ✓ next=plan_next_stop | 4.07 |
| 9 | plan_next_stop | ✓ next=plan_next_stop | 7.91 |
| 10 | plan_next_stop | ✓ next=plan_next_stop | 0.09 |
| 11 | plan_next_stop | ✓ next=plan_next_stop | 0.83 |
| 12 | plan_next_stop | ✓ next=plan_next_stop | 10.41 |
| 13 | plan_next_stop | ✓ next=plan_next_stop | 0.58 |
| 14 | plan_next_stop | ✓ next=plan_next_stop | 0.66 |
| 15 | plan_next_stop | ✓ next=plan_next_stop | 0.12 |
| 16 | plan_next_stop | ✓ next=plan_next_stop | 0.77 |
| 17 | plan_next_stop | ✓ next=plan_next_stop | 7.79 |
| 18 | plan_next_stop | ✓ next=plan_next_stop | 0.68 |
| 19 | plan_next_stop | ✓ next=plan_next_stop | 0.65 |
| 20 | plan_next_stop | ✓ next=plan_next_stop | 0.09 |
| 21 | plan_next_stop | ✓ next=plan_next_stop | 4.0 |
| 22 | plan_next_stop | ✓ next=plan_next_stop | 10.44 |
| 23 | plan_next_stop | ✓ next=plan_next_stop | 3.84 |
| 24 | plan_next_stop | ✓ next=trip_complete | 7.53 |

## 结果：成功（trip_complete）

**Trip Store:** `trip_id=cmudgeojd000602rgm5dn1fqy` · `revision=23`

## 骨架

- **Day 1** 贝伦区经典：Hills Hotel Lisboa → 贝伦塔 → lunch → 热罗尼莫斯修道院 → dinner
- **Day 2** 辛特拉山宫与童话森林：Hills Hotel Lisboa → 辛特拉 → lunch → 辛特拉 → dinner
- **Day 3** 卡斯凯什海滨与城堡：Hills Hotel Lisboa → 卡斯凯什 → lunch → 卡斯凯什 → dinner
- **Day 4** 里斯本老城历史高地：Hills Hotel Lisboa → 圣若热城堡 → lunch → 卡尔莫修道院 → dinner

## 逐站填充结果

### Hills Hotel Lisboa  · stay
- 时段：09:00 – 09:00
- 评分：4
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7303691%2C-9.1404614)
- [google_app](https://maps.google.com/?q=38.7303691%2C-9.1404614)
- [amap_web](https://uri.amap.com/marker?position=-9.1404614,38.7303691&name=Hills%20Hotel%20Lisboa)
- 备注：origin_stop

### 贝伦塔  · attraction
- 时段：10:17 – 11:02
- 到达：transit 约 77 分钟
- 起点直达：transit 约 77 分钟
- 备注：station_timing_adjusted

### Otsumami  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 9 分钟
- 评分：5
- 类别：japanese_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.694754499999995%2C-9.2156217)
- [google_app](https://maps.google.com/?q=38.694754499999995%2C-9.2156217)
- [amap_web](https://uri.amap.com/marker?position=-9.2156217,38.694754499999995&name=Otsumami)
- 备注：station_timing_adjusted

### 热罗尼莫斯修道院  · attraction
- 时段：13:17 – 14:02
- 到达：transit 约 47 分钟
- 备注：station_timing_adjusted

### Hino Café & Brunch  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 3 分钟
- 评分：4.9
- 类别：brunch_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.698995599999996%2C-9.2052813)
- [google_app](https://maps.google.com/?q=38.698995599999996%2C-9.2052813)
- [amap_web](https://uri.amap.com/marker?position=-9.2052813,38.698995599999996&name=Hino%20Caf%C3%A9%20%26%20Brunch)

### Hills Hotel Lisboa  · stay
- 时段：09:00 – 09:00
- 评分：4
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7303691%2C-9.1404614)
- [google_app](https://maps.google.com/?q=38.7303691%2C-9.1404614)
- [amap_web](https://uri.amap.com/marker?position=-9.1404614,38.7303691&name=Hills%20Hotel%20Lisboa)
- 备注：origin_stop

### 辛特拉  · attraction
- 时段：09:35 – 10:20
- 到达：drive 约 35 分钟
- 起点直达：drive 约 35 分钟
- 备注：station_timing_adjusted

### Sagres  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 1 分钟
- 类别：coffee_shop
- [google_web](https://www.google.com/maps/search/?api=1&query=38.8233416%2C-9.322921299999999)
- [google_app](https://maps.google.com/?q=38.8233416%2C-9.322921299999999)
- [amap_web](https://uri.amap.com/marker?position=-9.322921299999999,38.8233416&name=Sagres)
- 备注：station_timing_adjusted, meal_low_signal, transit_heuristic

### 辛特拉  · attraction
- 时段：13:03 – 13:48
- 到达：drive 约 33 分钟
- 备注：station_timing_adjusted

### Sagres  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 1 分钟
- 类别：coffee_shop
- [google_web](https://www.google.com/maps/search/?api=1&query=38.8233416%2C-9.322921299999999)
- [google_app](https://maps.google.com/?q=38.8233416%2C-9.322921299999999)
- [amap_web](https://uri.amap.com/marker?position=-9.322921299999999,38.8233416&name=Sagres)
- 备注：station_timing_adjusted, meal_low_signal, transit_heuristic

### Hills Hotel Lisboa  · stay
- 时段：09:00 – 09:00
- 评分：4
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7303691%2C-9.1404614)
- [google_app](https://maps.google.com/?q=38.7303691%2C-9.1404614)
- [amap_web](https://uri.amap.com/marker?position=-9.1404614,38.7303691&name=Hills%20Hotel%20Lisboa)
- 备注：origin_stop

### 卡斯凯什  · attraction
- 时段：10:35 – 11:20
- 到达：transit 约 95 分钟
- 起点直达：transit 约 95 分钟
- 备注：station_timing_adjusted

### Boutique del Jamón  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 9 分钟
- 评分：4.9
- 类别：tapas_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.6981931%2C-9.4350269)
- [google_app](https://maps.google.com/?q=38.6981931%2C-9.4350269)
- [amap_web](https://uri.amap.com/marker?position=-9.4350269,38.6981931&name=Boutique%20del%20Jam%C3%B3n)
- 备注：station_timing_adjusted

### 卡斯凯什  · attraction
- 时段：13:58 – 14:43
- 到达：transit 约 88 分钟
- 备注：station_timing_adjusted

### Boutique del Jamón  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 9 分钟
- 评分：4.9
- 类别：tapas_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.6981931%2C-9.4350269)
- [google_app](https://maps.google.com/?q=38.6981931%2C-9.4350269)
- [amap_web](https://uri.amap.com/marker?position=-9.4350269,38.6981931&name=Boutique%20del%20Jam%C3%B3n)
- 备注：station_timing_adjusted

### Hills Hotel Lisboa  · stay
- 时段：09:00 – 09:00
- 评分：4
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7303691%2C-9.1404614)
- [google_app](https://maps.google.com/?q=38.7303691%2C-9.1404614)
- [amap_web](https://uri.amap.com/marker?position=-9.1404614,38.7303691&name=Hills%20Hotel%20Lisboa)
- 备注：origin_stop

### 圣若热城堡  · attraction
- 时段：09:41 – 10:26
- 到达：walk 约 41 分钟
- 起点直达：walk 约 41 分钟
- 备注：station_timing_adjusted

### LUDO'S  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 5 分钟
- 评分：4.9
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7127792%2C-9.1322867)
- [google_app](https://maps.google.com/?q=38.7127792%2C-9.1322867)
- [amap_web](https://uri.amap.com/marker?position=-9.1322867,38.7127792&name=LUDO'S)

### 卡尔莫修道院  · attraction
- 时段：12:52 – 13:37
- 到达：walk 约 22 分钟
- 备注：station_timing_adjusted

### VeganBuffet  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 1 分钟
- 评分：4.8
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.7113964%2C-9.1402889)
- [google_app](https://maps.google.com/?q=38.7113964%2C-9.1402889)
- [amap_web](https://uri.amap.com/marker?position=-9.1402889,38.7113964&name=VeganBuffet)
- 备注：station_timing_adjusted
