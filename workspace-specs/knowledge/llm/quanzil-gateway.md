---
title: Quanzil OpenAI-compatible gateway
type: ops-lesson
status: active
as_of: 2026-08-17
tags:
  - quanzil
  - llm
  - openai-sdk
related_spec: workspace-specs/3.tech-specs.md
related:
  - knowledge/handbook.md
  - adr/ADR-004-quanzil-fixed-per-deployable.md
---

# Quanzil — OpenAI-compatible gateway (not api.openai.com)

## Summary

Product and agent LLM use **Quanzil** via the `openai` SDK. Env names stay `OPENAI_*` (ADR-004). The host is **Quanzil**, not OpenAI’s public API. Do not set `OPENAI_BASE_URL` to `https://api.openai.com/v1`.

## Evidence

- Architecture §5 / ADR-004: Quanzil per deployable; not destination-routed.
- Tech spec: `OPENAI_BASE_URL` inventory `https://quanzil.com/v1`; key console `https://quanzil.com/console/token`.
- Chat param: `max_completion_tokens` (not `max_tokens`) on this OpenAI-compatible surface.
- Optional inventory models (e.g. `gemini-2.5-flash` on the same gateway) are experiments, not a geo router.

## Lesson / guidance

| Concern | Rule |
| --- | --- |
| Client | `openai` SDK with `baseURL` = `OPENAI_BASE_URL` |
| Chat model | `OPENAI_CHAT_MODEL` (inventory default `gpt-5.4`) |
| Image model | `OPENAI_IMAGE_MODEL` when a product needs images (not places-agent MVP tools) |
| Where keys live | That deployable’s server / Portainer env. Never the browser. |
| Agent vs app | places-agent has its own `OPENAI_*`; what2eat / where2play BFF have theirs. |
| Failures | Return error to caller — never silent empty success |

Local filled inventory may live in `0.1.sdd.sample/.keys` (gitignored). Specs list env codes only.

## Links

- [ADR-004](../../adr/ADR-004-quanzil-fixed-per-deployable.md)
- [`3.tech-specs.md`](../../3.tech-specs.md)
