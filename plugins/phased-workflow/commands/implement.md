---
description: "Implement a phase from a plan file. Pass just a plan path to start Phase 1, or pass a plan path and phase number to implement that phase (all prior phases are inferred as complete). Use after clearing context."
model: claude-opus-4-6
argument-hint: "[path/to/PLAN.md] [phase number]"
---

Read the implementation plan at: $1

**Determine which phase to implement:**
- If a phase number was provided as "$2", implement **Phase $2** from this plan.
- If no phase number was provided (i.e. "$2" is empty or blank), implement **Phase 1** from this plan.

**Determine which prior phases are complete:**
- If implementing Phase 1, no prior phases exist.
- If implementing Phase N (where N > 1), all prior phases (1 through N-1) have already been implemented and exist in the codebase. Do NOT re-implement or modify work from prior phases unless the current phase explicitly requires it.

**Important context:**
- Each phase in this plan is self-contained. Everything you need to know is described within the phase itself.
- No prior phases are in your working memory. Rely only on the plan document and the current state of the codebase.
- You have access to the `neon-db` subagent if you need to query the database for schema info, test data, or to verify migrations.
- After implementation, verify the phase's acceptance criteria are met.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone

When complete, summarize:
1. What you implemented
2. Files created or modified
3. Whether all acceptance criteria passed
4. Any concerns or notes for subsequent phases
5. How many phases remain (e.g., "3 of 5 phases complete — 2 phases remaining")
