# E2E-04 罗马（Rome）3天行程

> 本文件由 `scripts/e2e-places-agent.py` 自动生成，模拟用户调用 places-agent 工具链路得到的真实结果。

## 模拟用户输入（8 行表单）

| 字段 | 值 |
| --- | --- |
| 城市 | 罗马（Rome） |
| 出发日期 | 2026-10-10 |
| 天数 | 3 |
| 酒店 | Hotel Vilon |
| 节奏 | relaxed |
| 预算 | 3（宽松） |
| 兴趣 | 古罗马遗迹、教堂 |
| 必去 | 梵蒂冈 |

## places-agent 工具链路

1. `geocode`（有酒店时）→ `travel_tips`（记录 `iconic_places`，ADR-045 展示源）→ 2. `discover_places` → 3. `make_itinerary` → 4. `plan_next_stop` 链直到 `trip_complete`（F65：无 `display_current_stop`）

## 工具调用记录

| # | 工具 | 结果 | 耗时(s) |
| --- | --- | --- | --- |
| 1 | geocode | ✓  | 0.38 |
| 2 | travel_tips | ✓ iconic= | 4.91 |
| 3 | discover_places | ✓ places=36, restaurants=40 | 9.46 |
| 4 | make_itinerary | ✓ next=plan_next_stop | 28.6 |
| 5 | plan_next_stop | ✓ next=plan_next_stop | 8.72 |
| 6 | plan_next_stop | ✓ next=plan_next_stop | 5.66 |
| 7 | plan_next_stop | ✓ next=plan_next_stop | 12.68 |
| 8 | plan_next_stop | ✓ next=plan_next_stop | 7.25 |
| 9 | plan_next_stop | ✓ next=plan_next_stop | 8.65 |
| 10 | plan_next_stop | ✓ next=plan_next_stop | 13.68 |
| 11 | plan_next_stop | ✓ next=plan_next_stop | 0.1 |
| 12 | plan_next_stop | ✓ next=plan_next_stop | 6.39 |
| 13 | plan_next_stop | ✓ next=plan_next_stop | 11.38 |
| 14 | plan_next_stop | ✓ next=plan_next_stop | 8.43 |
| 15 | plan_next_stop | ✓ next=plan_next_stop | 7.7 |
| 16 | plan_next_stop | ✓ next=plan_next_stop | 17.69 |
| 17 | plan_next_stop | ✓ next=plan_next_stop | 0.1 |
| 18 | plan_next_stop | ✓ next=plan_next_stop | 2.91 |
| 19 | plan_next_stop | ✓ next=plan_next_stop | 7.56 |
| 20 | plan_next_stop | ✓ next=plan_next_stop | 4.36 |
| 21 | plan_next_stop | ✓ next=plan_next_stop | 4.44 |
| 22 | plan_next_stop | ✓ next=trip_complete | 10.68 |

## 结果：成功（trip_complete）

**Trip Store:** `trip_id=cmudozugw003x02rggl3537ik` · `revision=21`

## 骨架

- **Day 1** 古罗马核心遗迹：Hotel Vilon → 罗马斗兽场 → lunch → 古罗马广场 → 君士坦丁凯旋门 → dinner
- **Day 2** 梵蒂冈圣城：Hotel Vilon → 梵蒂冈博物馆 → lunch → 西斯汀小堂 → 圣彼得大教堂 → dinner
- **Day 3** 巴洛克罗马与古典神殿：Hotel Vilon → 万神庙 → lunch → 胜利之后圣母堂 → 西班牙阶梯 → dinner

## 逐站填充结果

### Hotel Vilon  · stay
- 时段：09:00 – 09:00
- 评分：4.8
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=41.904466%2C12.476546599999999)
- [google_app](https://maps.google.com/?q=41.904466%2C12.476546599999999)
- [amap_web](https://uri.amap.com/marker?position=12.476546599999999,41.904466&name=Hotel%20Vil%C3%B2n)
- 备注：origin_stop

### 罗马斗兽场  · attraction
- 时段：09:37 – 10:22
- 到达：walk 约 37 分钟
- 起点直达：walk 约 37 分钟
- 备注：station_timing_adjusted

### Ristoro Della Salute  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 5 分钟
- 评分：4.8
- 类别：italian_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.8899271%2C12.4942499)
- [google_app](https://maps.google.com/?q=41.8899271%2C12.4942499)
- [amap_web](https://uri.amap.com/marker?position=12.4942499,41.8899271&name=Ristoro%20Della%20Salute)
- 备注：station_timing_adjusted

### 古罗马广场  · attraction
- 时段：12:42 – 13:27
- 到达：walk 约 12 分钟
- 备注：station_timing_adjusted

### 君士坦丁凯旋门  · attraction
- 时段：13:37 – 14:22
- 到达：walk 约 10 分钟
- 备注：station_timing_adjusted

### Ristoro Della Salute  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 6 分钟
- 评分：4.8
- 类别：italian_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.8899271%2C12.4942499)
- [google_app](https://maps.google.com/?q=41.8899271%2C12.4942499)
- [amap_web](https://uri.amap.com/marker?position=12.4942499,41.8899271&name=Ristoro%20Della%20Salute)
- 备注：station_timing_adjusted

### Hotel Vilon  · stay
- 时段：09:00 – 09:00
- 评分：4.8
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=41.904466%2C12.476546599999999)
- [google_app](https://maps.google.com/?q=41.904466%2C12.476546599999999)
- [amap_web](https://uri.amap.com/marker?position=12.476546599999999,41.904466&name=Hotel%20Vil%C3%B2n)
- 备注：origin_stop

### 梵蒂冈博物馆  · attraction
- 时段：09:35 – 10:20
- 到达：walk 约 35 分钟
- 起点直达：walk 约 35 分钟
- 备注：station_timing_adjusted

### Caffè Delle Commari  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 18 分钟
- 评分：4.8
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.908024%2C12.4540957)
- [google_app](https://maps.google.com/?q=41.908024%2C12.4540957)
- [amap_web](https://uri.amap.com/marker?position=12.4540957,41.908024&name=Caff%C3%A8%20Delle%20Commari)
- 备注：station_timing_adjusted

### 西斯汀小堂  · attraction
- 时段：13:13 – 13:58
- 到达：walk 约 43 分钟
- 备注：station_timing_adjusted

### 圣彼得大教堂  · attraction
- 时段：14:03 – 14:48
- 到达：walk 约 5 分钟

### Angelo's  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 9 分钟
- 评分：4.9
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.9000148%2C12.4550356)
- [google_app](https://maps.google.com/?q=41.9000148%2C12.4550356)
- [amap_web](https://uri.amap.com/marker?position=12.4550356,41.9000148&name=Angelo's)
- 备注：station_timing_adjusted

### Hotel Vilon  · stay
- 时段：09:00 – 09:00
- 评分：4.8
- 类别：hotel
- [google_web](https://www.google.com/maps/search/?api=1&query=41.904466%2C12.476546599999999)
- [google_app](https://maps.google.com/?q=41.904466%2C12.476546599999999)
- [amap_web](https://uri.amap.com/marker?position=12.476546599999999,41.904466&name=Hotel%20Vil%C3%B2n)
- 备注：origin_stop

### 万神庙  · attraction
- 时段：09:12 – 09:57
- 到达：walk 约 12 分钟
- 起点直达：walk 约 12 分钟
- 备注：station_timing_adjusted

### Osteria da Fortunata - Pantheon  · meal
- 时段：11:30 – 12:30
- 到达：walk 约 1 分钟
- 评分：4.8
- 类别：restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.8992918%2C12.476453399999999)
- [google_app](https://maps.google.com/?q=41.8992918%2C12.476453399999999)
- [amap_web](https://uri.amap.com/marker?position=12.476453399999999,41.8992918&name=Osteria%20da%20Fortunata%20-%20Pantheon)

### 胜利之后圣母堂  · attraction
- 时段：12:54 – 13:39
- 到达：walk 约 24 分钟
- 备注：station_timing_adjusted

### 西班牙阶梯  · attraction
- 时段：13:56 – 14:41
- 到达：walk 约 17 分钟
- 备注：station_timing_adjusted

### Alla Rampa  · meal
- 时段：17:30 – 19:00
- 到达：walk 约 4 分钟
- 评分：4.6
- 类别：italian_restaurant
- [google_web](https://www.google.com/maps/search/?api=1&query=41.905326099999996%2C12.483400099999999)
- [google_app](https://maps.google.com/?q=41.905326099999996%2C12.483400099999999)
- [amap_web](https://uri.amap.com/marker?position=12.483400099999999,41.905326099999996&name=Alla%20Rampa)
- 备注：station_timing_adjusted
