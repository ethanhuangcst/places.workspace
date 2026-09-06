# ADR-035: macOS places-agent 后台启动用 start_new_session 脱离

## Status
Accepted

## Context
places-agent 在 macOS + Cursor/IDE agent 短生命周期 shell 下反复出现：`/v1/health` 刚通过 → 数秒内 :3010 无人监听；`.data/server.pid` 指向已死进程。根因不是 Next/LLM 崩溃（无 fatal 栈），而是进程未真正脱离父会话。

`scripts/dev-up.sh` 优先走 `setsid`；**Darwin 无 `setsid`**，回退 `nohup … &`。实测 nohup 仍会被 agent/CI 结束会话时带走（与既有 ops 笔记一致）。另：`tsx watch` 与 `npx tsx server.ts` 多路启动留下孤儿 watch 进程；仅信 pid 文件会误报「已启动」。

## Decision
1. **macOS / 无 setsid 环境：** 后台启动必须用 **POSIX 新会话** 脱离（Python `subprocess.Popen(..., start_new_session=True)` 或等价 double-fork/`os.setsid`），不要依赖裸 `nohup &`。
2. **`make status` / 「已启动」判定：** 以 **LISTEN(PORT) + `GET /v1/health`** 为准；pid 文件仅辅助，失效则清理 pid 与 `.next/dev/lock`。
3. **日常开发：** 仍优先长生命周期前台 `make dev`；`make up` 仅在已按 (1) 硬脱离后视为可靠后台。
4. **停机：** `pkill` 模式限定本仓库路径，避免误杀其它项目的 `tsx … server.ts`。

## Rationale
- **start_new_session vs nohup：** nohup 只忽略 SIGHUP，不保证脱离 agent 的进程组/会话收割；`start_new_session` 创建新 session，与「health OK 后秒死」现象对齐。
- **health 为准 vs 信 pid：** 多次复现 stale pid + lock 仍写死 PID，导致假阳性。
- **不选 launchd 作为本地默认：** 过重；先脚本层脱离，生产仍用容器 restart policy。

## Consequences
- 已落地：`scripts/daemon_detach.py`（`start_new_session=True`）+ 薄封装 `dev-up.sh` / `dev-down.sh`；`make status` 以 LISTEN + `/v1/health` 为准。
- Agent 自动化可稳定拉起 :3010，减少「以为起了其实没起」。
- `dev-down` 的 `pkill` 限定本仓库路径，避免误杀其它项目的 `tsx … server.ts`。
- 单测：`make test-scripts`（`scripts/test_daemon_detach.py`）。

## Date
2026-08-22
