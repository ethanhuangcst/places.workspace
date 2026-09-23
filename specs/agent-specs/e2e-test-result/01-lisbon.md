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
| 1 | geocode | ✓  | 2.37 |
| 2 | travel_tips | ✓ iconic= | 4.89 |
| 3 | discover_places | ✓ places=51, restaurants=40 | 10.45 |
| 4 | make_itinerary | ✓ next=plan_next_stop | 29.68 |
| 5 | plan_next_stop | ✓ next=plan_next_stop | 3.49 |
| 6 | plan_next_stop | ✓ next=plan_next_stop | 3.03 |
| 7 | plan_next_stop | ✓ next=plan_next_stop | 14.42 |
| 8 | plan_next_stop | ✓ next=plan_next_stop | 3.3 |
| 9 | plan_next_stop | ✓ next=plan_next_stop | 18.36 |
| 10 | plan_next_stop | ✓ next=plan_next_stop | 0.1 |
| 11 | plan_next_stop | ✓ next=plan_next_stop | 0.95 |
| 12 | plan_next_stop | ✓ next=plan_next_stop | 58.41 |
| 13 | plan_next_stop | ✓ next=plan_next_stop | 0.6 |
| 14 | plan_next_stop | ✓ next=plan_next_stop | 0.93 |
| 15 | plan_next_stop | ✓ next=plan_next_stop | 0.09 |
| 16 | plan_next_stop | ✓ next=plan_next_stop | 0.98 |
| 17 | plan_next_stop | ✓ next=plan_next_stop | 16.61 |
| 18 | plan_next_stop | ✓ next=plan_next_stop | 0.51 |
| 19 | plan_next_stop | ✓ next=plan_next_stop | 0.92 |
| 20 | plan_next_stop | ✓ next=plan_next_stop | 0.08 |
| 21 | plan_next_stop | ✓ next=plan_next_stop | 7.82 |
| 22 | plan_next_stop | ✓ next=plan_next_stop | 10.7 |
| 23 | plan_next_stop | ✓ next=plan_next_stop | 5.03 |
| 24 | plan_next_stop | ✓ next=trip_complete | 1.25 |

## 结果：成功（trip_complete）

**Trip Store:** `trip_id=cmudovq78001u02rge5n37sdf` · `revision=23`

## 骨架

- **Day 1** 贝伦区经典：Hills Hotel Lisboa → 贝伦塔 → lunch → 热罗尼莫斯修道院 → dinner
- **Day 2** 辛特拉山林宫殿：Hills Hotel Lisboa → 辛特拉 → lunch → 辛特拉 → dinner
- **Day 3** 卡斯凯什海岸风情：Hills Hotel Lisboa → 卡斯凯什 → lunch → 卡斯凯什 → dinner
- **Day 4** 里斯本市中心历史与观景：Hills Hotel Lisboa → 圣若热城堡 → lunch → Miradouro das Portas do Sol → dinner

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
- 时段：09:57 – 10:42
- 到达：transit 约 57 分钟
- 起点直达：transit 约 57 分钟
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
- 时段：13:07 – 13:52
- 到达：transit 约 37 分钟
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
- 时段：10:48 – 11:33
- 到达：transit 约 108 分钟
- 起点直达：transit 约 108 分钟
- 备注：station_timing_adjusted

### Sagres  · meal
- 时段：11:45 – 12:45
- 到达：walk 约 1 分钟
- 类别：coffee_shop
- [google_web](https://www.google.com/maps/search/?api=1&query=38.8233416%2C-9.322921299999999)
- [google_app](https://maps.google.com/?q=38.8233416%2C-9.322921299999999)
- [amap_web](https://uri.amap.com/marker?position=-9.322921299999999,38.8233416&name=Sagres)
- 备注：station_timing_adjusted, meal_low_signal, transit_heuristic

### 辛特拉  · attraction
- 时段：14:32 – 15:17
- 到达：transit 约 107 分钟
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
- 时段：10:22 – 11:07
- 到达：transit 约 82 分钟
- 起点直达：transit 约 82 分钟
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
- 时段：13:47 – 14:32
- 到达：transit 约 77 分钟
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

### Miradouro das Portas do Sol  · attraction
- 时段：12:56 – 13:41
- 到达：walk 约 25 分钟
- 备注：station_timing_adjusted

### Osteria Bellosguardo  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 2 分钟
- 评分：4.8
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=38.711718499999996%2C-9.1295518)
- [google_app](https://maps.google.com/?q=38.711718499999996%2C-9.1295518)
- [amap_web](https://uri.amap.com/marker?position=-9.1295518,38.711718499999996&name=Osteria%20Bellosguardo)
- 备注：station_timing_adjusted
