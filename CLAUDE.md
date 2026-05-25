# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo contains Claude Code plugins and skills. Currently it has one plugin: `phased-workflow`, a structured development workflow codified as slash commands.

## Plugin Structure

Plugins live in named subdirectories and follow this layout:

```
<plugin-name>/
  .claude-plugin/
    plugin.json       # Plugin metadata (name, description, version, author)
  commands/
    <command>.md      # One file per slash command
  README.md
```

### Command File Format

Each command `.md` file has a YAML frontmatter block followed by the prompt body:

```yaml
---
description: "..."
allowed-tools: Read, Grep, Glob   # optional — restricts available tools
argument-hint: "[usage hint]"      # optional — shown in autocomplete
model: claude-opus-4-6             # optional — pins a model; omit to let users run their own
---
```

- `$ARGUMENTS` — injects the full argument string passed by the user
- `$1`, `$2`, etc. — injects individual positional arguments

## phased-workflow Plugin

Four commands that enforce a disciplined build cycle:

| Command | Slash Command | Purpose |
|---|---|---|
| `brainstorm.md` | `/phased-workflow:brainstorm` | Read-only discussion; no code edits allowed |
| `plan.md` | `/phased-workflow:plan` | Creates `./PLAN.md` with numbered, self-contained phases |
| `implement.md` | `/phased-workflow:implement <PLAN.md> [phase#]` | Implements one phase; defaults to Phase 1 |
| `validate.md` | `/phased-workflow:validate <PLAN.md>` | Validates full implementation against the plan |

**Key design constraint:** Each phase in a plan must be self-contained — implementable with no memory of prior phases. The implement command treats all prior phases as already complete in the codebase.

**Cross-phase memory:** To bridge the `/clear` between phases, `implement` maintains a companion scratchpad named `<plan-name>-history.md` (derived from the plan's filename). Each phase reads it for carried-forward context and appends a concise note of its own. `validate` reads it to focus its review, then deletes it once validation passes cleanly (it's retained if CRITICAL issues are found). The plan document remains the source of truth; the history file only supplements it.

**Recommended workflow:** brainstorm → plan → `/clear` → implement phase 1 → `/clear` → implement phase 2 → ... → `/clear` → validate
