---
title: OPENAI_CN OpenAI-compatible gateway
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - openai-cn
  - llm
  - openai-sdk
related_spec: workspace-specs/3.tech-specs.md
related:
  - knowledge/handbook.md
  - adr/ADR-004-quanzil-fixed-per-deployable.md
  - adr/ADR-041-openai-cn-replaces-quanzil.md
---

# OPENAI_CN — OpenAI-compatible gateway (not api.openai.com)

Formerly called **Quanzil** (gateway retired). Specs use the name **OPENAI_CN**; env codes stay `OPENAI_*` ([ADR-041](../../adr/ADR-041-openai-cn-replaces-quanzil.md), [ADR-004](../../adr/ADR-004-quanzil-fixed-per-deployable.md)).

## Summary

Product and agent LLM use **OPENAI_CN** via the `openai` SDK. Do **not** set `OPENAI_BASE_URL` to `https://api.openai.com/v1`. **Do not hardcode** the gateway host in specs — each deployable sets `OPENAI_BASE_URL` / `OPENAI_HOST` / keys in server env.

## Evidence

- Architecture §5 / ADR-004: one OPENAI_CN config per deployable; not destination-routed.
- Chat param: `max_completion_tokens` (not `max_tokens`) on this OpenAI-compatible surface.
- Optional inventory models on the same gateway are experiments, not a geo router.

## Lesson / guidance

| Concern | Rule |
| --- | --- |
| Client | `openai` SDK with `baseURL` = `OPENAI_BASE_URL` |
| Chat model | `OPENAI_CHAT_MODEL` |
| Image model | `OPENAI_IMAGE_MODEL` when a product needs images (not places-agent MVP tools) |
| Host / base URL | From deployable env only — never bake a vendor hostname into umbrella specs |
| Key console | Optional `OPENAI_KEY_MGMT_SITE` |
| Where keys live | That deployable’s server / Portainer. Never the browser. |
| Agent vs app | places-agent has its own `OPENAI_*`; what2eat / where2play BFF have theirs. |
| Failures | Return error to caller — never silent empty success |

Local filled inventory may live in `0.1.sdd.sample/.keys` (gitignored). Specs list env codes only.

## Links

- [ADR-041](../../adr/ADR-041-openai-cn-replaces-quanzil.md)
- [ADR-004](../../adr/ADR-004-quanzil-fixed-per-deployable.md)
- [`3.tech-specs.md`](../../3.tech-specs.md)
