# ADR-009: Deploy Option 1 on 野草云3

## Status

Accepted

## Decision

Three services, three images, prefer three Portainer stacks on 野草云3 (`svr_hk_vps_3`). Pipeline: GHCR → Portainer → Nginx Proxy Manager → Cloudflare, following release-bot patterns. Avoid multi-process single images.

## Consequences

Independent deploy/rollback; ops aligned with existing VPS automation.
