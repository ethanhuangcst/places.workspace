# ADR-046: Trip Store（PG 权威 + 内存热副本）与宿主按需读取

## Status

**Accepted**（2026-09-02）— 决议来自产品方逐项确认；实现按 `1.places-agent/agent-specs/0.refactor-plan.md` 批次 **MVP-16** 分期落地。  
取代并关闭原 `agent-specs/TBD-1.md`（已删除）。

## Context

现行行程状态由宿主经 `next_tool_call` 回传大包 JSON（`skeleton` + `cursor` + candidates）。多工具协作时：

1. 拼装/传输成本高，宿主上下文臃肿。
2. 无法在服务端统一改「同一份」行程（换餐、调序、局部改骨架时往往要重跑 `make_itinerary`）。
3. 宿主读行程只能依赖工具返回的大包，缺少按字段切片的读模型。

MCP 传输保持 **stateless**（[ADR-045](./ADR-045-iconic-places-unified-acquisition.md)）。业务行程状态 ≠ MCP session。

探针结论（原 TBD-1 议题 B）：在现网关下「按日并发 LLM 骨架」平均不缩短墙钟；**不**作为默认路径。

## Decision

### D1 — 做 Trip Store

服务端维护一份权威行程账本（Trip）。各工具读写账本局部，不以整包 JSON 当共享内存。

### D2 — 双主目的

1. 多工具改**同一份**行程（换餐、调序、受控改骨架，不必整次重跑 `make_itinerary`）。
2. 本地宿主（尤其 where2play）能更好使用行程数据与上下文。

减 JSON 体积是附带收益，不是为再抠 LLM 秒数。

### D3 — 读路径：经 agent/BFF，不下发 DB 连接

宿主持 `trip_id`，经 places-agent（或 where2play BFF）按需取片段。  
**否决**把数据库连接串交给 Cursor/ChatBox 等宿主。BFF 持库仍属服务端读，不算宿主直连。

### D4 — 存储：PostgreSQL 权威 + 进程内内存热副本

- **权威：** PostgreSQL（[ADR-025](./ADR-025-places-agent-postgres-prisma.md) `places_agent` 库）。
- **热副本：** agent 进程内与 PG 同步的内存实体。
- **同步语义（非双主）：**
  - 写：改内存 → 落 PG → 成功后向宿主返回新 `revision`（可带 patch）。
  - 读：内存命中；未命中或 `revision` 落后则从 PG 灌回内存。
  - 冲突：乐观锁 `revision`；旧版本写失败则重载再试。

可选短缓存，不作第二真相源。P0 **不**引入 Redis。

### D5 — `trip_id` 懒创建（不新增 `start_trip`）

工具已偏多，不新增专用创建工具。  
任一需要行程账本的业务工具在**无** `trip_id` 时懒创建（PG + 内存），响应返回 `trip_id`，并要求宿主后续一律带上。调用顺序可变（discover / 天气 / visa / make 谁先谁建）。

### D6 — 宿主镜像策略

- **where2play（可控）：** 本地按同一 schema 全量（或近全量）hydrate。
- **Cursor / ChatBox（不可控）：** 弱镜像（`trip_id` + 摘要/光标）；细节 `fetch_trip_details`。

### D7 — 写后同步：`revision` + patch

写工具返回 `trip_id` + `revision` + 变更片段。可控宿主 apply patch；冲突或丢步再 fetch。默认不每次回传整本行程。

### D8 — 工具面（精简优先）

| 工具 | 决议 |
| --- | --- |
| `plan_next_stop` | **保留**（写侧按站推进） |
| `fetch_trip_details` | **新增**（只读；`trip_id` + `fields[]`） |
| `display_current_stop` | **尽快删除**；写职责并入 `plan_next_stop`；读一律 fetch |
| `patch_skeleton` | **仅内部** core 方法；不暴露 MCP/HTTP |
| `start_trip` / `create_trip` | **不新增** |

另：**评估并精简**现有对外工具（`arrange_day`、别名、过时搜索入口等），与删 display 同批规划，净增工具数尽量 ≤1（仅 `fetch_trip_details`）。

### D9 — 受控改骨架

允许内部 `patchSkeleton` 在规则内改骨架（如 meal 时段、微调顺序），经 `plan_next_stop` 等写路径触发；升 `revision` 并通过 patch 通知宿主。禁止静默大改。

### D10 — 与 ADR-045

Trip 是业务状态：须 `trip_id` 显式、TTL/`expires_at`、`caller_key` 隔离；找不到时 `trip_not_found`；`host_instructions` 禁止编造行程。不恢复 MCP transport session。

### D11 — 按日并发骨架（议题 B）

**默认不切换。** 保持一次全局 LLM 出多日骨架。并发仅可作后续可选实验，不阻塞本 ADR。

### D12 — 分期（实现指引）

见 refactor-plan **MVP-16**：

1. **P0：** PG 表 + 内存热副本；懒创建；写双写 + `revision`/patch；`fetch_trip_details`；对外工具精简评估清单落地项  
2. **P1：** 删 `display_current_stop`；写并入 `plan_next_stop`；fill 主路径 `trip_id`+光标  
3. **P2：** 内部受控 `patch_skeleton`；visa/天气/tips 挂 artifacts；TTL/鉴权收干净  
4. **P3（可选）：** 过期清理与观测  

## Consequences

### Positive

- 多工具共享权威行程；宿主上下文可变瘦。
- where2play 可稳定 hydrate；通用 MCP 宿主仍可用弱镜像。
- 与现有 Postgres 运维对齐；重启后 `trip_id` 仍可恢复（在 TTL 内）。

### Negative / risks

- 新表与 migrate；多实例下须依赖 PG + revision，内存仅本机加速。
- 删 `display_current_stop` 为破坏性变更：须同步 e2e、host_instructions、where2play plan-46。
- 懒创建若宿主未保存 `trip_id` 会造出多份空 trip——须文档与响应强提示。

### Rejected alternatives

- 宿主直连 DB（安全与 MCP 现实不可接受）。
- 仅内存权威（多实例/重启丢态）。
- 真·双主双向同步（无单一权威）。
- 新增 `start_trip`（工具膨胀）。
- 对外暴露 `patch_skeleton`。
- 默认按日并发 LLM 骨架。

## References

- [ADR-025](./ADR-025-places-agent-postgres-prisma.md) places-agent PostgreSQL  
- [ADR-045](./ADR-045-iconic-places-unified-acquisition.md) MCP stateless  
- [0.refactor-plan.md](../../1.places-agent/agent-specs/0.refactor-plan.md) MVP-16  
- Orizn 签证仍走 REST：[ADR-044](./ADR-044-orizn-visa-rest-adapter.md)（与 Cursor IDE MCP 无关）
