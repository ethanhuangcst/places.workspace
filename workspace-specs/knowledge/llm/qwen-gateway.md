---
title: Qwen primary LLM (Aliyun Bailian compatible-mode)
type: ops-lesson
status: active
as_of: 2026-09-03
tags:
  - qwen
  - llm
  - openai-sdk
related_spec: workspace-specs/3.tech-specs.md
related:
  - knowledge/handbook.md
  - adr/ADR-047-qwen-primary-llm.md
  - knowledge/llm/openai-cn-gateway.md
---

# Qwen — 主 LLM（阿里云百炼 compatible-mode）

[ADR-047](../../adr/ADR-047-qwen-primary-llm.md)：places-agent 与产品 BFF 的 **主** chat / 结构化生成走 **Qwen**。OPENAI_CN（`OPENAI_*`）仅为 `QWEN_API_KEY` 未设时的回退。旧笔记见 [`openai-cn-gateway.md`](./openai-cn-gateway.md)。

## Summary

`openai` SDK（或 BFF `fetch`）→ `QWEN_BASE_URL` + `QWEN_API_KEY` + `QWEN_CHAT_MODEL`（默认 `qwen-plus`）。不要用 `api.openai.com`。

本工作空间约定（host 来自操作者工作空间，可随租户变；**密钥不入库文档**）：

| Code | 约定值 / 形态 |
| --- | --- |
| `QWEN_HOST` | `{workspace}.cn-beijing.maas.aliyuncs.com` |
| `QWEN_BASE_URL` | `https://{host}/compatible-mode/v1` |
| `QWEN_NATIVE_BASE_URL` | `https://{host}/api/v1`（主路径不用） |
| `QWEN_REGION` | `cn-beijing` |
| `QWEN_CHAT_MODEL` | `qwen-plus` |
| `QWEN_CHAT_MODEL_FALLBACK` | `qwen-flash / qwen-turbo`（预留） |
| `QWEN_IMAGE_MODEL` | `qwen-image-2.0` |
| `QWEN_KEY_MGMT_SITE` | 百炼控制台（对应 region） |

## Lesson / guidance

| Concern | Rule |
| --- | --- |
| Client | `openai` SDK `baseURL` = `QWEN_BASE_URL` |
| Chat model | `QWEN_CHAT_MODEL` |
| Image | `QWEN_IMAGE_MODEL` when a product needs images |
| Keys | That deployable’s `.env.local` / Portainer. Never the browser. |
| Agent vs app | Each of places-agent, what2eat, where2play has its own `QWEN_*`. |
| Fallback | Empty `QWEN_API_KEY` → existing `OPENAI_*` |
| Failures | Return error to caller — never silent empty success |

Local inventory template: `0.1.sdd.sample/.keys`（gitignored）。Specs 只列 env codes。

## Links

- [ADR-047](../../adr/ADR-047-qwen-primary-llm.md)
- [`3.tech-specs.md`](../../3.tech-specs.md)
