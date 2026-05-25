---
description: "Implement a phase from a plan file. Pass just a plan path to start Phase 1, or pass a plan path and phase number to implement that phase (all prior phases are inferred as complete). Use after clearing context."
argument-hint: "[path/to/PLAN.md] [phase number]"
---

Read the implementation plan at: $1

**Determine which phase to implement:**
- If a phase number was provided as "$2", implement **Phase $2** from this plan.
- If no phase number was provided (i.e. "$2" is empty or blank), implement **Phase 1** from this plan.

**Determine which prior phases are complete:**
- If implementing Phase 1, no prior phases exist.
- If implementing Phase N (where N > 1), all prior phases (1 through N-1) have already been implemented and exist in the codebase. Do NOT re-implement or modify work from prior phases unless the current phase explicitly requires it.

**Load cross-phase memory (do this before implementing):**
- Derive the history file path from the plan path "$1" by replacing the trailing `.md` with `-history.md` (e.g. `.../add-user-auth.md` → `.../add-user-auth-history.md`).
- If that file exists, read it. It contains concise notes the prior phases chose to carry forward — decisions made and why, deviations from the plan, gotchas, and interfaces/names that later phases must match.
- Treat it as supplementary context that bridges the `/clear` between phases. The plan document remains the source of truth; if the history and the plan ever conflict, follow the plan.
- If the file does not exist (e.g. this is Phase 1), just proceed.

**Important context:**
- Each phase in this plan is self-contained. Everything you need to know is described within the phase itself.
- No prior phases are in your working memory. Rely on the plan document, the history file (if present), and the current state of the codebase.
- You have access to the `neon-db` subagent if you need to query the database for schema info, test data, or to verify migrations.
- After implementation, verify the phase's acceptance criteria are met.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone

**Record cross-phase memory (do this after implementing, before summarizing):**
- Append a section for this phase to the history file (the `-history.md` path derived above). Create the file if it does not yet exist; its first line should be `# <plan filename> — implementation history`.
- Use this exact heading for the appended section: `## Phase <N> — <short phase title>`.
- Under it, write a SHORT bulleted list (aim for 3–7 bullets, skip any that don't apply) capturing only what a future phase genuinely needs and could NOT trivially recover from the plan or the codebase:
  - Decisions you made where the plan left a choice open, and why.
  - Deviations from the plan, and the reason.
  - Names/signatures/interfaces/paths later phases must match exactly.
  - Gotchas or surprises encountered (setup quirks, non-obvious dependencies, things that broke).
  - Follow-ups or tech debt deliberately deferred.
- Be token-efficient. This is a scratchpad, not documentation — omit anything obvious, restated from the plan, or already evident in the code. If nothing is worth carrying forward, append the heading with a single bullet saying so.

When complete, summarize:
1. What you implemented
2. Files created or modified
3. Whether all acceptance criteria passed
4. Any concerns or notes for subsequent phases
5. What you recorded to the history file (one line)
6. How many phases remain (e.g., "3 of 5 phases complete — 2 phases remaining")
