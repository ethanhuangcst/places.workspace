# Nominate probe — Hangzhou AMAP + Lisbon GMAP

Generated: 2026-09-09T05:50:12.110Z
Model: `qwen-plus`
Bounds: 2026-07-01..2026-07-04 · 4 days · couple_romance
Ground: suggest → hydrate → search → broad-fallback (strip suffix, >15km). Cap cluster 3. native_id dedupe.
Quality: chips≥5, ground≥0.6, no 残雪 in LLM names, no junk chips, ≥1 chip >20km from city anchor when numDays≥3 (distance gate; clusterMax display-only).
Overall: **FAIL**

## 杭州 · AMAP

- Dates in prompt: **yes**
- Season word: **yes**
- Names: 10 · grounded rate: **100%** (need ≥60%)
- Chips: **7** (need ≥5)
- 残雪 in names: **no**
- Junk chips: **no**
- Max same cluster (display): **1**
- Suburb chip >20km from anchor (numDays≥3): **NO**
- Gate: **FAIL**

### Prompt

```
为以下行程推荐必去地。只输出 JSON 字符串数组，每项是可在地图搜索框原样搜到的短地名（不要括号、不要说明、不要开头结尾散文）。不要安排行程，不要编造坐标或商家编号。写寺、塔、桥、园等专名，不要只写整座湖、整条街、整座新城。不要线路（如电车线路）、活动、打包描述。符合出行月份与季节，不要列该季看不到或不宜游的季节专属景。目的地近郊有可安排一日游/半日游的知名必去地时，应至少列入一个。不要全集中在同一片区域（如同一湖周边、同一街区）。
杭州 4天 2026-07-01至2026-07-04 7月 夏季 2人 情侣浪漫 舒适 适中节奏 打车优先
```

### Parsed names

1. 灵隐寺
2. 雷峰塔
3. 断桥
4. 西湖十景-曲院风荷
5. 西溪国家湿地公园
6. 宋城
7. 河坊街
8. 六和塔
9. 中国美术学院象山校区
10. 龙井村

### Grounding (suggest→search)

| # | nominate | hit | vendor name | chip label |
| --- | --- | --- | --- | --- |
| 1 | 灵隐寺 | yes | 灵隐寺 | 灵隐寺 |
| 2 | 雷峰塔 | yes | 雷峰塔景区 | 雷峰塔 |
| 3 | 断桥 | yes | 断桥残雪 | 断桥 |
| 4 | 西湖十景-曲院风荷 | yes | 曲院风荷 | 西湖十景-曲院风荷 |
| 5 | 西溪国家湿地公园 | yes | 西溪国家湿地公园 | 西溪国家湿地公园 |
| 6 | 宋城 | **no** | — | — |
| 7 | 河坊街 | **no** | — | — |
| 8 | 六和塔 | yes | 六和塔 | 六和塔 |
| 9 | 中国美术学院象山校区 | **no** | — | — |
| 10 | 龙井村 | yes | 龙井村 | 龙井村 |

### Chips (label = nominated short name)

1. 灵隐寺 _(vendor: 灵隐寺)_
2. 雷峰塔 _(vendor: 雷峰塔景区)_
3. 断桥 _(vendor: 断桥残雪)_
4. 西湖十景-曲院风荷 _(vendor: 曲院风荷)_
5. 西溪国家湿地公园 _(vendor: 西溪国家湿地公园)_
6. 六和塔 _(vendor: 六和塔)_
7. 龙井村 _(vendor: 龙井村)_

<details><summary>raw</summary>

```
["灵隐寺", "雷峰塔", "断桥", "西湖十景-曲院风荷", "西溪国家湿地公园", "宋城", "河坊街", "六和塔", "中国美术学院象山校区", "龙井村"]
```

</details>

## Lisbon · GMAP

- Dates in prompt: **yes**
- Season word: **yes**
- Names: 8 · grounded rate: **100%** (need ≥60%)
- Chips: **8** (need ≥5)
- 残雪 in names: **no**
- Junk chips: **no**
- Max same cluster (display): **1**
- Suburb chip >20km from anchor (numDays≥3): **yes**
- Gate: **PASS**

### Prompt

```
Recommend must-see places for this trip. Output a JSON array of short place names only — the same words a map search box can use. No parentheses, no explanations, no preamble or footer. Do not build an itinerary. Do not invent coordinates or place IDs. Use specific venue names (tower, monastery, castle, park gate), not a whole lake, street, or new-town district. No routes, events, or parenthetical bundles. Fit the travel month and season; do not list seasonal-only sights that are not typical or not visible then. When the destination has well-known nearby places reachable as a day or half-day trip, include at least one. Do not cluster all in one neighborhood or around one landmark.
Lisbon 4 days 2026-07-01 to 2026-07-04 2026-07 summer 2 people Couple romance Comfort Balanced Taxi first
```

### Parsed names

1. Belém Tower
2. Jerónimos Monastery
3. Alcázar of Segovia
4. Castelo de São Jorge
5. Praça do Comércio
6. Parque das Nações
7. Sintra National Palace
8. Queluz National Palace

### Grounding (suggest→search)

| # | nominate | hit | vendor name | chip label |
| --- | --- | --- | --- | --- |
| 1 | Belém Tower | yes | Belém Tower | Belém Tower |
| 2 | Jerónimos Monastery | yes | Jerónimos Monastery | Jerónimos Monastery |
| 3 | Alcázar of Segovia | yes | Alcázar de Segovia | Alcázar of Segovia |
| 4 | Castelo de São Jorge | yes | Castelo de São Jorge | Castelo de São Jorge |
| 5 | Praça do Comércio | yes | Praça do Comércio | Praça do Comércio |
| 6 | Parque das Nações | yes | Parque das Nações | Parque das Nações |
| 7 | Sintra National Palace | yes | National Palace of Sintra | Sintra National Palace |
| 8 | Queluz National Palace | yes | Queluz National Palace | Queluz National Palace |

### Chips (label = nominated short name)

1. Belém Tower _(vendor: Belém Tower)_
2. Jerónimos Monastery _(vendor: Jerónimos Monastery)_
3. Alcázar of Segovia _(vendor: Alcázar de Segovia)_
4. Castelo de São Jorge _(vendor: Castelo de São Jorge)_
5. Praça do Comércio _(vendor: Praça do Comércio)_
6. Parque das Nações _(vendor: Parque das Nações)_
7. Sintra National Palace _(vendor: National Palace of Sintra)_
8. Queluz National Palace _(vendor: Queluz National Palace)_

<details><summary>raw</summary>

```
["Belém Tower", "Jerónimos Monastery", "Alcázar of Segovia", "Castelo de São Jorge", "Praça do Comércio", "Parque das Nações", "Sintra National Palace", "Queluz National Palace"]
```

</details>
