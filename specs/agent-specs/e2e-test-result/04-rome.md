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
| 1 | geocode | ✗ Expecting value: line 1 column 1 (char 0) |  |

## 结果：失败

```
geocode failed: Expecting value: line 1 column 1 (char 0)
```
