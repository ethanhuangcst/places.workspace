---
title: AMAP 20 城 text / weight 探针（discover 方案可行性）
type: ops-lesson
status: active
as_of: 2026-09-06
tags:
  - amap
  - discover
  - quality
related:
  - knowledge/maps/amap-around-distance-quality.md
  - knowledge/maps/amap-cn20-probe.json
  - adr/ADR-042-no-city-encyclopedia-in-source.md
---

# AMAP 20 城 text / weight 探针

## Summary

原方案「discover 改 `/v5/place/text`、去掉 L0 裸 `景区`」**可行，但必须带 `types=110000`（风景名胜）**。裸 `keywords=景点` + `region` 在杭州第一页仍无断桥/雷峰塔，且并发易触发 `CUQPS_HAS_EXCEEDED_THE_LIMIT` (10021)。

20 城 live（2026-09-06）：`text + 景点 + types=110000 + region + city_limit` + 拟议 L0 后，**iconic 信号（塔/寺/堤/桥/祠/宫/苑/陵/窟/墙）≥3 的城：12/20**；现网 around+distance+现行 L0：**2/20**。around 改 `sortrule=weight` 为 **9/20**（杭州同样出断桥+雷峰塔）。正餐 `text + 餐厅 + types=050000` 商场店占比普遍低于 1 km around。

探针城名单与脚本 **不是** 产品 CATALOG（ADR-042）。

## Method

| 臂 | API | L0 |
|----|-----|-----|
| Baseline | around，`keywords=景点`，15 km，`sortrule=distance` | 现行（含裸 `景区`） |
| Weight | around 同上，`sortrule=weight` | 拟议（去裸 `景区`；剥末尾 `景区`；unwrap `名胜区-子点`） |
| Text | `/v5/place/text`，`keywords=景点`，`types=110000`，`region=城`，`city_limit=true` | 拟议 |
| 餐 around | around 1 km，`餐厅` + `050000` + distance | — |
| 餐 text | text，`餐厅` + `050000` + region | — |

脚本：`places-agent/scripts/amap-cn20-text-vs-around-probe.mjs`。原始 JSON：`amap-cn20-probe.json`。iconic 正则是探针启发式，会漏掉拙政园/留园这类「园」；看样本名。

## Headline counts

| 臂 | 城数 iconic≥3 | iconic≥5 |
|----|----------------|----------|
| around + distance + 现行 L0 | 2 | — |
| around + weight + 拟议 L0 | 9 | — |
| text + 110000 + 拟议 L0 | **12** | 4 |

杭州：baseline iconic **0**（Do都城 / 市民广场一类）；text / weight 均 **断桥残雪、雷峰塔、吴山**。正餐 around 商场占比 **0.50**（市民中心食堂/肯德基）；text **0.00**（杭州酒家、黄龙海鲜等；同参另一次第 7 条为楼外楼）。

## 样本（text 臂首屏）

| 城 | text 拟议 L0 首屏 | iconic≥3? |
|----|-------------------|-----------|
| 北京 | 天安门广场、故宫、天坛、景山、北海 | 是 |
| 上海 | 外滩、豫园、东方明珠、城隍庙、静安寺 | 是 |
| 杭州 | 清河坊、断桥残雪、吴山、湖滨公园、雷峰塔 | 是 |
| 西安 | 大明宫、汉城湖、城墙、广仁寺 | 是 |
| 成都 | 铁像寺、双子塔、锦城公园（少见大熊猫基地） | 是 |
| 重庆 | 洪崖洞、十八梯、魁星楼、罗汉寺 | 是 |
| 南京 | 鸡鸣寺、玄武湖、总统府、夫子庙、城墙、明孝陵 | 是 |
| 苏州 | 山塘、寒山寺、留园、平江路、拙政园、虎丘 | 否*（园未计入正则，质量好） |
| 广州 | 越秀、大佛寺、光孝寺、陈家祠、六榕寺、广州塔 | 是 |
| 深圳 | 莲花山、笔架山、华侨城（无塔寺桥信号） | 否 |
| 武汉 | 江滩、江汉关、中山公园 | 否 |
| 长沙 | 开福寺、橘子洲头、岳麓书院 | 否（正则只计寺） |
| 厦门 | 南普陀寺、植物园、筼筜湖（鼓浪屿在未剥父名时可能被拒） | 是 |
| 青岛 | 五四广场、湛山寺、栈桥 | 是 |
| 昆明 | 公园/斗南为主，无大观楼/滇池首屏 | 否 |
| 桂林 | 象鼻山、独秀峰、日月双塔 | 否（差 1） |
| 哈尔滨 | 冰雪大世界、太阳岛、防洪纪念塔 | 是 |
| 天津 | 水上公园、西开教堂（少五大道/瓷房子） | 否 |
| 郑州 | 公园/二七（少嵩山/少林，符合市区 region） | 否 |
| 沈阳 | 沈阳故宫、弥陀寺 | 是 |

## Lesson

1. **可行主路径：** discover 大陆建池用 text + `types=110000` + `region` + `city_limit`；正餐建池用 text + `050000`。fill 走廊仍 around。
2. **不要**只改「不用 around」而不加 types：杭州 `景点` text 第一页是步行街/灯光秀。
3. **around + weight** 是较小改动的次优（杭州已够用，20 城略弱于 text）。
4. L0 去裸 `景区` + 剥后缀 **必要**：否则 `雷峰塔景区`/`吴山景区` 仍进不了池。
5. 深圳/武汉/昆明/天津 text 仍偏公园 —— 不靠城表补点；热度或第二关键词另开故事。
6. QPS：20 城并行会 10021；产品路径并发已有上限，保持。
7. 禁止把本探针的城→POI 表抄进源码。

## Links

- [amap-around-distance-quality.md](./amap-around-distance-quality.md)
- 原始表：[amap-cn20-probe.json](./amap-cn20-probe.json)
