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
| 2 | travel_tips | ✓ iconic= | 4.43 |
| 3 | discover_places | ✗ Expecting value: line 1 column 1 (char 0) |  |

## 结果：失败

```
discover_places failed: Expecting value: line 1 column 1 (char 0)
```
