# code_agent

Personal, general-purpose config for AI coding agents (Claude Code, Codex).
Cherry-picked copies — not the live config, and nothing here replaces `~/.claude` or `~/.codex`.

## Layout

| Path | What goes here |
|---|---|
| `instructions/` | Agent-neutral instructions (`AGENTS.md`) |
| `skills/` | Skills usable by any agent (`<name>/SKILL.md`) |
| `claude/` | Claude Code only: `CLAUDE.md`, `settings.example.json`, `agents/`, `bin/`, `commands/`, `hooks/`, `output-styles/`, `skills/` |
| `codex/` | Codex only: `config.example.toml`, `hooks.json`, `agents/`, `rules/`, `skills/` |

## Rules

- General-purpose only — no employer-internal skills, systems, or data.
- No secrets. Real `settings.json` / `config.toml` never land here; commit a scrubbed `*.example.*` instead.
- A skill goes in `skills/` unless it depends on one agent's features; then it goes under that agent.
