---
description: "Enter plan mode and create a phased implementation plan with self-contained phases. Use after brainstorming."
argument-hint: "[description of what to plan]"
---

Enter plan mode. I need you to create a detailed implementation plan for the following:

$ARGUMENTS

**Requirements for the plan:**

1. Break the implementation into self-contained phases. Each phase MUST be implementable without knowledge of prior phases in active working memory. This means each phase should include:
   - Clear description of what it accomplishes
   - All file paths that will be created or modified
   - Complete context needed (no "as we did in phase 1" references — spell it out)
   - Acceptance criteria for when the phase is complete (written per #2)
   - A `Verification:` line (per #3)
   - Any database schema changes or migrations needed

2. **Write acceptance criteria as checkable assertions, not prose.** Every criterion must be something that can be mechanically confirmed: a command to run and its expected result, a route to load and what must appear on it, a query and the rows it should return. Do not write "works correctly", "handles errors gracefully", or "is properly integrated" — those cannot be verified, so they get rubber-stamped.

3. **Tag every phase with a `Verification:` line** naming exactly one of:
   - `unit` — the behavior is provable by tests. Name the specific test files to create or update.
   - `e2e` — the phase touches UI, a user-facing flow, or a full request → DB → response path. Name the route(s) to visit, the actions to perform, and the observable outcome that proves success. These get driven in a real browser during implementation.
   - `none` — no behavioral change (copy tweak, constant, comment, pure rename). State in one line why nothing needs verifying. Use sparingly; if in doubt, it is not `none`.

   Determine the project's test and lint commands from `package.json` / `CLAUDE.md` and name them in the plan, so each phase can be checked without rediscovery.

4. Number the phases sequentially (Phase 1, Phase 2, etc.)

5. Include a summary section at the top with:
   - Total number of phases
   - High-level goal of each phase (one line each)
   - Estimated complexity per phase (low/medium/high)
   - Verification type per phase (unit/e2e/none)

6. Save the plan as a markdown file at: /Users/brianwhite/.claude/plans/<descriptive-filename>.md
   - The filename should be a short, kebab-case description of what the plan covers (e.g., `add-user-auth.md`, `refactor-api-routes.md`)
   - Do NOT use generic names like `PLAN.md` or `plan.md`

7. Do NOT implement anything. Only create the plan document.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone

After creating the plan, tell me:
- The filepath where you saved it
- How many phases the plan has
- Any phases that feel risky or complex
- The verification mix (how many unit / e2e / none). If most phases are `none`, that is a signal the plan is under-specified — say so.
- **Whether any phases are genuinely independent of one another.** Phases run sequentially by default. If a group of them shares no files, no schema changes, and no interfaces, say so explicitly and note that I could run them in parallel with `/batch` instead of one at a time. Only raise this when the independence is real — a wrong call here costs more than it saves.

I will review the plan, then clear context and begin implementation phase by phase.
