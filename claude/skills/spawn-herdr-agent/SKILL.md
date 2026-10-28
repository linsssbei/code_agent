---
name: spawn-herdr-agent
description: "Run a subagent as a visible Claude agent in this session's Herdr subagents tab, so the user can watch it work. Use INSTEAD of the built-in Agent tool whenever you would spawn a subagent and HERDR_ENV=1, or when the user runs /spawn-herdr-agent. Requires Herdr; falls back to the built-in Agent tool outside Herdr."
argument-hint: "<task brief>"
---

# Spawn a Herdr subagent

Environment check (runs before you read this):

!`if [ "${HERDR_SUBAGENT:-}" = 1 ]; then echo "herdr=nested"; elif [ "${HERDR_ENV:-}" = 1 ] && command -v herdr >/dev/null 2>&1; then echo "herdr=available"; else echo "herdr=unavailable"; fi`

- `herdr=unavailable`: Herdr is not running here. Use the built-in Agent tool as usual and ignore the rest of this skill.
- `herdr=nested`: you are a subagent yourself. Do not spawn Herdr subagents; do the work inline or use the built-in Agent tool.
- `herdr=available`: follow the steps below.

The helper is `~/.claude/bin/herdr-subagent`. It owns all layout and lifecycle rules: one tab per Claude session, up to 8 panes, recycle the oldest *collected* idle slot when full. Do not run raw `herdr` commands to create, close, or recycle subagent panes.

## Steps

1. **Write a self-contained brief.** The subagent starts with zero context from this conversation: state the goal, relevant paths, what is already ruled out, constraints (e.g. read-only), and the exact answer shape. Ask for the final answer as the last message. If the task is user-invoked, the brief is `$ARGUMENTS` plus any context it needs.
2. **Spawn** (one call per subagent; independent spawns can go in parallel Bash calls):
   ```bash
   ~/.claude/bin/herdr-subagent spawn "<brief>"
   ```
   It prints the agent name (e.g. `ccsub-83a3f4af-03`). Tell the user the name; they can watch it in the `ccsub-<session[:8]>` tab.
3. **Collect** when you need the result. This waits, prints the agent's recent screen, and marks the slot recyclable:
   ```bash
   ~/.claude/bin/herdr-subagent collect <name> --timeout 600000 --lines 200
   ```
   Set the Bash tool timeout above the `--timeout` value. If the answer is cut off, re-run with more `--lines`. For long outputs, have the brief ask the subagent to write Markdown to `/tmp/<name>.md` and read that file.
4. **Report** the subagent's result to the user. Its pane stays open until recycled or cleared with `/clear-idle-herdr-agent`.

## Exit codes

| Code | Meaning | Action |
|---|---|---|
| 3 | All 8 slots are working, blocked, or uncollected | Collect finished ones, or tell the user and suggest `/clear-idle-herdr-agent`. Do not close panes yourself. |
| 4 | Called from inside a subagent | Do the work inline. |
| 5 | `collect` timed out or agent not settled | Run `herdr agent get <name>` and `herdr agent read <name>`. If `blocked`, it is waiting on a permission prompt: tell the user which agent; do not answer it for them. Do not resubmit the brief. |

`~/.claude/bin/herdr-subagent status mine` lists this session's subagents (`status` lists every session's).

Ownership is recorded on each pane as Herdr metadata tokens (`role=ccsub`, `owner`, `session`, `seq`, `collected`). Never write those tokens yourself.
