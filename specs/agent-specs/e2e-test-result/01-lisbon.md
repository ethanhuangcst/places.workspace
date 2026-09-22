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
| 1 | geocode | ✓  | 1.37 |
| 2 | travel_tips | ✓ iconic= | 10.56 |
| 3 | discover_places | ✓ places=47, restaurants=40 | 14.69 |
| 4 | make_itinerary | ✓  | 30.61 |

## 结果：失败

```
chain ended without trip_complete after 0 calls
```

**Trip Store:** `trip_id=cmuchxqjy000602juv7p5b0y4` · `revision=3`

## 骨架

- **Day 1** 贝伦区经典：Hills Hotel Lisboa → 贝伦塔 → lunch → 热罗尼莫斯修道院 → dinner
- **Day 2** 辛特拉山宫与童话景观：Hills Hotel Lisboa → 辛特拉 → lunch → Monserrate Palace Ticket office → dinner
- **Day 3** 卡斯凯什海滨与城堡：Hills Hotel Lisboa → 卡斯凯什 → lunch → Palácio da Cidadela de Cascais → dinner
- **Day 4** 里斯本老城历史与观景台：Hills Hotel Lisboa → 圣若热城堡 → lunch → Miradouro das Portas do Sol → dinner
