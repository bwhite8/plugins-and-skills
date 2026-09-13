---
description: "Execute an entire plan end-to-end: implement every phase in its own fresh subagent, verify each, stop on failure, then validate and run review passes. Use after /phased-workflow:plan."
argument-hint: "[path/to/PLAN.md] [phase number to resume from]"
---

Read the implementation plan at: $1

You are the **orchestrator** for this plan. You will NOT implement any phase yourself — each phase runs in its own fresh subagent so that no phase inherits another phase's context. Your job is to dispatch, gate, and report. Keep your own context lean: do not read the source files each phase touches.

**Determine the run range:**
- Count the phases in the plan. Call this N.
- If a phase number was provided as "$2", start from **Phase $2** (phases before it are already complete). Otherwise start from **Phase 1**.

**Preflight — do this before dispatching anything:**
1. Derive the history file path from "$1" by replacing the trailing `.md` with `-history.md`.
2. Scan every phase in the run range for operations that need my approval: deleting files or directories, dropping or truncating tables, destructive migrations, or anything else not easily undone. **Surface all of them to me now, in one list, and wait for my go-ahead.** Approving them here covers the whole run — the phase subagents should not stop to re-ask. If there are none, say so and proceed without waiting.
3. Report the plan's verification mix (how many `unit` / `e2e` / `none` phases). If most phases are `none`, warn me the plan is under-specified before burning a run on it.

**Phase loop — for each phase K from the start phase through N, one at a time, in order:**

Phases share files and build on each other, so they run **sequentially**, never in parallel. Dispatch each one as a subagent and wait for its result before starting the next.

Use the **`phased-workflow:phase-implementer`** agent type. Its operating rules — scope discipline, verification requirements, the history-file contract, and the report format — live in its system prompt, so keep the per-phase prompt short. Give it only:
- The plan path "$1" and the derived history file path
- "Implement Phase K." — nothing more; the agent knows what that entails
- "Destructive operations described in the plan are pre-approved by the user for this run."

**If that agent type is unavailable** (plugin not loaded, resolution fails), fall back to `general-purpose`, say so plainly in your output, and add the rules to the prompt yourself — the fallback agent has none of them:

> Read the plan and the history file first; where they conflict, the plan wins. Implement Phase K only — prior phases are already implemented, do not modify them, do not start the next. Verify before claiming done: run the project's test and lint commands whatever the phase's tag; for `unit`, add the named test files and confirm they would fail without the new code; for `e2e`, drive the app in a real browser via the Playwright MCP tools (`mcp__plugin_playwright_playwright__browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, `browser_console_messages`, `browser_network_requests`), confirming the phase's stated outcome and checking console and network for new errors. Re-reading your own code is not verification — anything unconfirmed goes in an explicit Unverified list. Append `## Phase K — <short title>` to the history file with 3–7 tight bullets (decisions where the plan left a choice, deviations and why, interfaces later phases must match, gotchas, tests and fixtures added, deferred or unverified work). Return concise prose — no code listings.

**Gate after each phase — this is the important part:**
- If the subagent reports test or lint failures it did not resolve, a failed acceptance criterion, or anything critical in its Unverified list: **stop the run immediately.** Do not dispatch the next phase. Report which phase stopped it, the specific failure, and tell me I can resume with `/phased-workflow:run $1 <K>` once it is fixed.
- A pre-existing failure the subagent proves is unrelated to its phase is not a stop condition — note it and continue.
- After each phase, print one line: `Phase K/N — <title> — <passed | stopped>` so I can watch progress.

**Final validation — only after every phase in the range completes:**

Run the full validation pass (the same work `/phased-workflow:validate $1` does — do it inline here rather than telling me to run it separately):
1. Project test and lint commands, results reported first.
2. Completeness check across every phase's acceptance criteria, weighted toward anything the history file flagged as unverified.
3. Re-exercise all `e2e` flows **in sequence** in a real browser via the Playwright MCP tools — integration bugs live in the seams between phases, not inside them.
4. Integration check: broken cross-phase references, inconsistent interfaces, schema mismatches.
5. `/code-review` over the full diff for this plan's work (plain local review — the `ultra` variant is user-launched only; recommend it to me if the diff warrants it). Then `/security-review` over the same diff, weighted toward auth boundaries, admin-only routes, raw SQL interpolation, and secret handling. If either is unavailable in this session, do the equivalent review inline and say which you substituted.
6. Residual scan for what a diff-scoped review cannot see: cross-phase race conditions and data-integrity problems, plus edge cases the plan implies that no test or browser check exercises.

**History cleanup:** if validation passed with NO CRITICAL issues, delete the `-history.md` file and confirm you did. If there are CRITICAL issues, retain it and say why. Never delete the plan itself or any other file.

**Final report:**
- `Phase K/N` line for every phase, with how it was verified
- Test and lint results
- Issues as CRITICAL / WARNING / INFO, noting which came from `/code-review` or `/security-review`
- An explicit **Unverified** list — anything not confirmed, and why
- Total phases completed vs. planned
- Do not call the implementation "complete" or "production-ready" if anything sits in the Unverified list or any CRITICAL issue is open. Say what is left instead.
