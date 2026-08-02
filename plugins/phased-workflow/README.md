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

### 3. Run the whole plan (recommended)

```
/phased-workflow:run ~/.claude/plans/<plan-name>.md
```

Executes the entire plan end-to-end. Each phase runs in **its own fresh subagent**, so phases stay context-isolated exactly as if you had `/clear`-ed between them — but without the manual re-invocation.

Run `/clear` once before this, right after the plan is written. The plan lives on disk, so nothing is lost, and the orchestrator starts lean. Do **not** clear between `brainstorm` and `plan` — the tradeoffs worked out while brainstorming only exist in conversation, and the plan depends on them.

The run:

1. **Preflights** — surfaces every destructive operation across all phases in one list and waits for a single go-ahead, then reports the plan's verification mix.
2. **Implements each phase sequentially**, verifying as it goes: tests and lint on every phase, new tests for `unit` phases, a real browser click-through via Playwright MCP for `e2e` phases.
3. **Gates after each phase** — a failed check stops the run immediately rather than cascading breakage into later phases. Resume with `/phased-workflow:run <plan> <phase number>` once fixed.
4. **Validates at the end** — full completeness and integration check, `e2e` flows re-exercised in sequence, then `/code-review` and `/security-review` over the diff.

To resume a stopped run, or to skip phases already done:

```
/clear
/phased-workflow:run ~/.claude/plans/<plan-name>.md 3
```

Clear first when resuming after a long debugging detour — the plan and the `-history.md` file carry everything the completed phases established, so the debugging transcript is only weighing the orchestrator down.

### Manual phase-by-phase (alternative)

Use this when you want to inspect the work between phases, or when a phase needs hands-on iteration.

```
/clear
/phased-workflow:implement ~/.claude/plans/<plan-name>.md
```

Implements Phase 1 (defaults to Phase 1 when no number is given). Then, for each remaining phase:

```
/clear
/phased-workflow:implement ~/.claude/plans/<plan-name>.md 2
```

Clearing context between phases is what keeps each one working from a clean slate — the plan document and the history file carry everything forward, not conversation memory.

When all phases are done:

```
/clear
/phased-workflow:validate ~/.claude/plans/<plan-name>.md
```

Validates the whole implementation: completeness, cross-phase integration, `e2e` flows re-run in sequence, and delegated `/code-review` and `/security-review` passes.

## Verification

Every phase carries a `Verification:` tag assigned at plan time, and acceptance criteria are written as checkable assertions (a command and its expected result, a route and what must appear on it) rather than prose like "works correctly."

| Tag | Meaning | How it is checked |
| --- | --- | --- |
| `unit` | Behavior is provable by tests | Named test files created/updated and run; a test that would pass without the new code doesn't count |
| `e2e` | Touches UI, a user flow, or a full request → DB → response path | Driven in a real browser via Playwright MCP — routes visited, actions performed, console and network checked for new errors |
| `none` | No behavioral change (copy, constant, rename) | Tests and lint only |

Tests and lint run on **every** phase regardless of tag. Re-reading the code that was just written does not count as verification, and anything that couldn't be confirmed is reported in an explicit **Unverified** list rather than folded into "all criteria passed."

## The phase-implementer Agent

`run` dispatches each phase to a dedicated **`phased-workflow:phase-implementer`** agent that ships with this plugin (`agents/phase-implementer.md`). Its scope discipline, verification requirements, history-file contract, and report format live in its system prompt rather than being re-injected into every phase prompt — so a ten-phase run states them once instead of ten times, and the rules sit where phase content can't crowd them out.

The agent deliberately does not pin a model or restrict its tools: it inherits the session model (better than hardcoding a tier that will be wrong for either the trivial or the hard phases), and it needs a broad tool surface anyway — file edits, shell for tests and lint, Playwright MCP for browser checks, and the `neon-agent` subagent for database verification.

If the agent type cannot be resolved, `run` falls back to `general-purpose`, inlines a condensed version of the rules, and says so in its output rather than silently running an undisciplined agent.

## Cross-Phase Memory

Because each phase is implemented in an isolated context — a fresh subagent under `run`, or a fresh window after `/clear` when going manually — the workflow keeps a lightweight scratchpad to bridge phases: a companion file next to the plan named `<plan-name>-history.md` (e.g. `add-user-auth.md` → `add-user-auth-history.md`).

- **During `implement`**: before working, Claude reads the history file (if present) for context the prior phases chose to carry forward. After working, it appends a short, token-efficient note for the current phase — decisions made, deviations from the plan, names/interfaces later phases must match, and gotchas. The plan stays the source of truth; the history file only supplements it.
- **During `validate`**: Claude reads the history file to focus its review on recorded deviations and gotchas, then deletes it as cleanup once validation passes. If validation surfaces CRITICAL issues, the file is retained so you can fix and re-validate with the notes intact.

The history file is auto-managed — you don't need to create or edit it yourself.

## Autonomous Execution

All commands run autonomously by default — Claude will proceed with actions without pausing to ask "should I proceed?" or "shall I make this change?" at each step. This keeps the workflow moving efficiently.

**Mandatory exceptions:** Claude will always ask before performing destructive operations that cannot be easily undone, such as deleting files/directories or dropping/truncating database tables.

## Why This Workflow?

- **Context independence**: Each phase can be implemented without prior phases in working memory, making it resilient to context window limits.
- **Structured planning**: Forces upfront thinking about phases, dependencies, and acceptance criteria.
- **Systematic validation**: Catches integration issues and bugs that can slip through phase-by-phase implementation.
- **Repeatable**: The same workflow works for any feature, regardless of complexity.
