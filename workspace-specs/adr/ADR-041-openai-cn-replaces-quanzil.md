# ADR-041: LLM 网关称呼 Quanzil → OPENAI_CN

## Status
Accepted（2026-08-23）

## Context
Quanzil（`quanzil.com`）已停服。伞仓与 knowledge 仍大量写「Quanzil」及 inventory `https://quanzil.com/v1`，与现行部署漂移。统一 **对外称呼**，且 **不在文档写死** 网关域名。

相关： [ADR-004](./ADR-004-quanzil-fixed-per-deployable.md)。

## Decision

### D1 — 用语（**已同意**）

1. 全局文档中产品/网关称呼：**Quanzil → OPENAI_CN**。  
2. **环境变量名不变**：`OPENAI_API_KEY` / `OPENAI_BASE_URL` / `OPENAI_CHAT_MODEL` 等。  
3. 禁止再写现行 inventory 为 `quanzil.com`；禁止默认 `api.openai.com`。  
4. 历史 ADR **文件名**可保留；正文注明「旧称 Quanzil，现称 OPENAI_CN」。  
5. knowledge：`llm/quanzil-gateway.md` → `llm/openai-cn-gateway.md`（旧路径短跳转）。

### D2 — inventory Base URL（**已同意 · 选 C**）

文档 **不写死** 网关域名。只写：各部署体在服务器 env 填入 OPENAI_CN 的 `OPENAI_BASE_URL`（及 key/model）。示例用占位或「见部署 env」，不用 `quanzil.com` / 不把某一商用域名写成规范默认。

### D3 — 改写范围（**已同意**）

| 范围 | 本故事 |
| --- | --- |
| `workspace-specs/**` 与各子仓 `*-specs/**` | **是** |
| `.env` / `.env.local` / `.env.example` 等 | **否** — 由操作者自改 |
| 运行时代码 fallback URL | **本故事不做**（随代码故事另开，或操作者保证 env 已设） |

### D4 — ADR-004

Accepted 正文用语改为 OPENAI_CN；文件名可暂保留。

## Consequences

- 规格读者看到的网关名是 OPENAI_CN；具体 host 只来自部署 env。  
- 历史文件名含 `quanzil` 的 ADR/knowledge 以正文更正为准。

## Date
2026-08-23
