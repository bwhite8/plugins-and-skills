# Phased Workflow Plugin

A Claude Code plugin that codifies a structured development workflow into slash commands. Ensures complex features are planned, implemented in self-contained phases, and validated systematically.

## Workflow

### 1. Brainstorm

```
/phased-workflow:brainstorm [your idea]
```

Discuss and validate the feasibility of an idea. No code edits — just a back-and-forth conversation about approach, risks, and tradeoffs.

### 2. Plan

```
/phased-workflow:plan [description of what to build]
```

Creates a detailed, phased implementation plan saved to `~/.claude/plans/<descriptive-name>.md`. The filename is auto-generated based on the plan contents. Each phase is fully self-contained so it can be implemented in a fresh context window.

### 3. Clear Context

```
/clear
```

Clear the conversation context before starting implementation. This ensures each phase is implemented with a clean slate, relying only on the plan document and codebase.

### 4. Implement Phase 1

```
/phased-workflow:implement ~/.claude/plans/<plan-name>.md
```

Implements Phase 1 from the plan. When no phase number is provided, defaults to Phase 1.

### 5. Clear and Implement Subsequent Phases

```
/clear
/phased-workflow:implement ~/.claude/plans/<plan-name>.md 2
```

Clear context, then implement the next phase. All prior phases are inferred as complete — the command relies on the plan document and current codebase state, not conversation memory.

Repeat steps 5 for each remaining phase (3, 4, etc.).

### 6. Validate

```
/clear
/phased-workflow:validate ~/.claude/plans/<plan-name>.md
```

After all phases are implemented, validate the entire implementation against the plan. Checks completeness, integration, and scans for critical bugs.

## Autonomous Execution

All commands run autonomously by default — Claude will proceed with actions without pausing to ask "should I proceed?" or "shall I make this change?" at each step. This keeps the workflow moving efficiently.

**Mandatory exceptions:** Claude will always ask before performing destructive operations that cannot be easily undone, such as deleting files/directories or dropping/truncating database tables.

## Why This Workflow?

- **Context independence**: Each phase can be implemented without prior phases in working memory, making it resilient to context window limits.
- **Structured planning**: Forces upfront thinking about phases, dependencies, and acceptance criteria.
- **Systematic validation**: Catches integration issues and bugs that can slip through phase-by-phase implementation.
- **Repeatable**: The same workflow works for any feature, regardless of complexity.
