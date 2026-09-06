# ADR-044: places-agent 接入 Orizn 签证数据（REST adapter + `visa_requirement` 工具）

## Status
Accepted（2026-09-01）— MVP-11 Feature 48 实现中

## Context

where2play 规划中的「出行建议页」需要按用户国籍给出目的地签证要求。数据源选定 [Orizn Visa](https://visa.orizn.app)（47,362 护照-目的地对、199 护照、238 目的地、15 语言；alpha-3 国家码）。places-agent 作为家族地点/行程网关（ADR-005 caller-driven 多供应商网关），签证是目的地级旅行事实，应由 agent 统一持有 vendor 密钥并对 caller 暴露工具，浏览器不持有 vendor key（ADR-002）。

Orizn 提供三种接入方式，2026-09-01 实测：

| 接入方式 | 实测结果 |
| --- | --- |
| 远程 MCP `https://visa.orizn.app/mcp` | **即使带 API Key 也只暴露 `quick_visa_check`**（签证类型 + 免签天数一行答案）；完整工具不可用；每 IP 30 对/天 keyless |
| 本地 stdio 子进程 `npx orizn-visa-mcp` | 完整 6 工具（`check_visa_requirement` 等）；但引入子进程 / npx 运行时依赖，容器部署与测试隔离更重 |
| **REST API `GET /api/v1/visa`** | 同一把 `x-api-key` 返回**与 stdio MCP 完整工具相同的数据**（requirement / 免签天数 / 中文材料清单 / 流程 / 审理时间 / 费用 / 有效期 / 停留 / 延期规则 / safety / 核验日期 / 官方来源 URL）；已用 CHN→JPN/KOR/SGP 验证 |

已有 adapter 模式先例：`src/adapters/{google,amap,tripadvisor,open-meteo}/` 均为 REST 直连 + `config.ts` env 加载 + `direct.ts` 客户端 + `fixture.ts` 测试夹具 + `live.ts` 组装（ADR-021 live 模式不得返回 fixture）。Open-Meteo 先例表明：行程内部的辅助数据源（非 `providers[]` 地图供应商）直接作为内部 adapter 接入即可，不进 caller 可选供应商列表。

Orizn 免费档约束：100 请求/月（未确认邮箱 5 次）；付费 Hobby $9/月 10,000 次。数据并非逐日核验——每对带 `last_verified` 日期与可选官方来源 URL，部分深层字段（费用、过境签等）在免费档返回 `{upgrade: "..."}` 占位。

## Decision

### D1 — REST 直连 adapter，不引入 MCP 子进程

新建 `src/adapters/orizn/`（`config.ts` / `direct.ts` / `fixture.ts` / `live.ts` / `card-mapper.ts` 等价物），遵循现有 adapter 模式：

- **env：** `ORIZN_API_KEY`（必填，live 模式）、`ORIZN_VISA_BASE_URL`（默认 `https://visa.orizn.app/api/v1`）、请求超时。密钥只存在于 places-agent 服务端，经 `.env.local` 运营商维护；不进浏览器、不进 client storage。
- **传输：** `GET /api/v1/visa?passport={alpha3}&destination={alpha3}&lang={code}`，`x-api-key` 头；注入式 `FetchFn` + AbortController 超时（与 `direct.ts` 模式一致）。
- **fixture：** `PLACES_VENDOR_MODE=fixture` 时返回固定样本对（CHN→JPN visa_required / CHN→SGP visa_free 30 天等），Fast CI 不消耗配额。

不采用 stdio 子进程：数据面完全相同，而 REST 与现有 adapter/测试/部署形态一致（agent-builder：工具接真实 API；不为此新增运行时依赖）。

**定位（与 Open-Meteo 同类）：** Orizn 是行程/目的地的**辅助数据源**，不进 `providers[]` 地图供应商机制、不参与 ADR-026 区域路由。`providers[]` 语义保持地图供应商不变。

### D2 — 工具面：`visa_requirement`（HTTP + MCP 双传输，ADR-003）

在 `src/core/tools.ts` 增加 `visaRequirement(input)`，经 `src/mcp/create-server.ts` 注册 MCP 工具、`src/http/dispatch.ts` 暴露 HTTP `POST /v1/visa_requirement`（同一工具核心，两传输对等）。

**输入 schema：**

```ts
{
  passport: string,      // ISO 3166-1 alpha-3，如 CHN
  destination: string,   // ISO 3166-1 alpha-3，如 JPN
  locale?: "EN"|"CN"|"HK"|"TW"  // 缺省 EN；映射 Orizn lang（EN→en, CN→zh, HK/TW→zh）
}
```

**输出 envelope（`ok`/`data`，与现有 jsonResult 一致）：**

```ts
{
  passport, destination,
  requirement: "visa_free"|"visa_required"|"e_visa"|"visa_on_arrival"|"eta"|"no_admission"|...,
  visa_free_days: number | null,
  description?: string,          // locale 化概述
  documents?: string[],          // 所需材料
  process?: string[],            // 申请/入境流程
  processing_time?: string,      // 审理时间
  validity?: string, max_stay?: string,
  extension?: { possible: boolean, details?: string },
  last_verified?: string | null, // 数据核验日期
  source_url?: string | null,    // 官方来源（如有）
  unavailable_fields?: string[]  // 免费档 upgrade 占位字段名（诚实降级）
}
```

- `lang` 映射：Orizn 无 `zh-HK`/`zh-TW`，HK/TW 映射 `zh`（与 agent 四 locale 输出契约不冲突——Orizn 返回的简中正文原样透传；家族 i18n 契约约束的是 agent 自产文案）。
- alpha-3 校验失败返回结构化错误（i18n key），不静默猜国家。

### D3 — 配额诚实与缓存（关键约束）

1. **配额保护缓存：** agent 进程内按 `(passport, destination, lang)` 短 TTL 缓存（签证规则变更频率低；建议 TTL 24h，可 env 调整 `ORIZN_CACHE_TTL_H`）。同对重复查询不重复消耗配额。
2. **配额耗尽 fail-closed 诚实：** Orizn 403/429（`quota_exceeded` / `missing_key`）→ 工具返回结构化 `outcome: "skipped"` 风格错误 + 明确 i18n key（如 `errors.visa_quota_exceeded`），**绝不**用编造签证信息冒充成功（与 ADR-021 live-honest、测试计划「不编造」条款一致）。
3. **upgrade 占位透明：** 免费档未开放的深层字段列入 `unavailable_fields`，不删除、不伪造值。
4. **数据非实时声明：** 输出携带 `last_verified`；工具 description 写明「以目的地官方移民机构为准」。

### D4 — where2play 侧：本切片只落国籍字段与规格，展示留出行建议页

- 本切片（spec-only）：注册页 + 个人资料页增加**国籍下拉**（`nationality`，存 ISO alpha-3；选填），Prisma `User.nationality String?` + 迁移；四 locale i18n key。
- 签证查询展示属**规划中的出行建议页 / Plan 贴士签证卡**：BFF 调 `visa_requirement` **写入** Trip；2play **经 `fetch_trip_details` 展示**。本切片 where2play 侧**不开发**查询 UI。
- 国籍下拉选项列表来自国家码标准表（ISO 3166-1 alpha-3），**非**目的地城市百科（ADR-042 不适用——这是用户属性输入控件，不是按城市硬编码 POI 策略）；但实现时选项文案须走 i18n，不得在源码内嵌单一语言国家名大表——使用 `Intl.DisplayNames`（运行时 locale 化）或 locale 资源文件，首选前者。

### D5 — 明确不做

- 不把签证状态写死进行程生成管线（plan_itinerary / arrange_day 不自动调 Orizn；由 caller/宿主显式调用 `visa_requirement`）。
- 不在 agent 内做 Orizn 之外的第二签证源抽象（单一 vendor，直连 adapter 即可；出现第二源时再议网关化）。
- 不做 Orizn `get_recent_changes` 消费（其官方 feed 已停用，返回空且明示不可据此断言无变化）。
- 远程 MCP 端点（keyless `quick_visa_check`）不作为生产数据面——功能不足以支撑材料/流程级答案。

## Consequences

- **正：** places-agent 获得签证工具，三个 caller（what2eat/where2play/ChatBox MCP 宿主）复用同一契约；vendor 密钥集中；fixture 模式零配额消耗；缓存把免费档 100/月放大为可有效支撑开发与演示。
- **负/风险：** Orizn 免费档配额小（100/月），live 调用须节制（缓存 + fail-closed）；数据核验频率有限（`last_verified` 透传给 caller 展示）；单 vendor 依赖，Orizn 停服则工具显式失败（诚实降级）。
- **中性：** `providers[]` 语义不变；HK/TW 的 Orizn 文案为简中（映射 zh）。
- **后续 ADR 触发点：** 若需第二签证源或把签证纳入行程自动编排，另立 ADR。

## 相关

- [ADR-002](./ADR-002-browser-bff-agent-trust.md)（浏览器不持 vendor key）
- [ADR-003](./ADR-003-dual-transport-one-core.md)（HTTP+MCP 同核）
- [ADR-005](./ADR-005-caller-provider-gateway.md)（caller-driven 网关；Orizn 不进 providers[]）
- [ADR-021](./ADR-021-live-vendor-no-fixture.md)（live 模式诚实性）
- [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)（国家码表 ≠ 城市百科；不冲突）
- agent-specs：`agent-stories.md` Feature 48 · `agent-design.md` §14 · `agent-test-plan.md` TC-M11-48
- 2play-specs：`2play-stories.md` Feature 38 · `2play-design.md` §2.6/§3.2/§3.4
