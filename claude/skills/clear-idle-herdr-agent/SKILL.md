---
name: clear-idle-herdr-agent
description: "Close idle Herdr subagent panes (shell included) created by /spawn-herdr-agent, across all Claude sessions by default, or only this session with `mine`. Keeps working/blocked agents and uncollected results whose owner session is still running. Requires Herdr."
argument-hint: "[mine]"
disable-model-invocation: true
---

# Clear idle Herdr subagents

Environment check (the skill aborts here outside Herdr):

!`if [ "${HERDR_ENV:-}" = 1 ] && command -v herdr >/dev/null 2>&1; then echo "herdr=available"; else echo "Herdr is not available in this session; nothing to clear."; exit 1; fi`

Scope argument: `$ARGUMENTS` (empty = every Claude session's subagents; `mine` = only this session's). Pass `mine` through only if the argument is exactly `mine`; otherwise pass nothing.

Rules, enforced by `~/.claude/bin/herdr-subagent clear`. A pane is a subagent only if its agent name is `ccsub-<owner[:8]>-NN`, its tokens say `role=ccsub`, its `session` token matches the session Herdr reports, and its tab is labelled `ccsub-<owner[:8]>`.

| Subagent | Owner session running | Owner session gone |
|---|---|---|
| collected, idle/done | close | close |
| uncollected, idle/done | keep | close |
| working / blocked | keep | keep |
| fails verification | keep | keep |

Agent-less panes in a `ccsub-` tab that carry `role=ccsub` tokens (failed spawns, exited agents) are closed. Closing a pane kills its shell; a tab disappears with its last pane.

## Steps

1. Show what will happen:
   ```bash
   ~/.claude/bin/herdr-subagent clear [mine] --dry-run
   ```
2. Run it:
   ```bash
   ~/.claude/bin/herdr-subagent clear [mine]
   ```
3. Report to the user: each closed agent with its `claude --resume <id>` line, and each kept agent with the reason.
