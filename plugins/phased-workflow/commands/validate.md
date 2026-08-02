---
description: "Validate that a completed plan was fully and accurately implemented with no critical bugs."
argument-hint: "[path/to/PLAN.md]"
---

Read the implementation plan at: $ARGUMENTS

This plan has been fully implemented across all phases. Your job is to validate the implementation.

**Load cross-phase memory:**
- Derive the history file path from the plan path "$ARGUMENTS" by replacing the trailing `.md` with `-history.md` (e.g. `.../add-user-auth.md` → `.../add-user-auth-history.md`).
- If that file exists, read it. It holds the notes each phase carried forward — decisions, deviations from the plan, and gotchas. Use it to focus your validation: deviations, gotchas, and anything recorded as unverified are prime spots to scrutinize.

**Perform the following validation:**

1. **Automated checks first**: run the project's test and lint commands (from `package.json` / `CLAUDE.md` — typically `npm run test` and `npm run lint`). Report results before anything else; a red suite frames everything that follows.

2. **Completeness check**: Go through every phase and verify that each acceptance criterion has been met. List any gaps. Pay particular attention to criteria the history file flagged as unverified.

3. **End-to-end check of `e2e` phases**: each phase tagged `Verification: e2e` was confirmed in isolation during implementation. Re-exercise those flows now **in sequence**, in a real browser, using the Playwright MCP tools (`mcp__plugin_playwright_playwright__browser_navigate`, `browser_snapshot`, `browser_click`, `browser_fill_form`, `browser_console_messages`, `browser_network_requests`). Integration bugs live in the seams between phases, not inside them. If those tools are unavailable, say so and mark the flows unverified rather than inferring from code.

4. **Integration check**: Verify that all phases work together correctly. Look for:
   - Missing imports or broken references between phases
   - Inconsistent naming or interfaces
   - Database schema mismatches

5. **Delegated review passes** — run these rather than reviewing ad hoc; they are purpose-built and more consistent than freehand inspection:
   - `/code-review` over the diff for this plan's work. Use the plain local review — the `ultra` variant is user-launched only, so recommend it to me if the diff warrants it rather than attempting to start it.
   - `/security-review` over the same diff. Weight it toward auth boundaries, admin-only routes, raw SQL interpolation, and secret handling.
   - If either command is unavailable in this session, perform the equivalent review inline and state which one you had to substitute.
   - Fold their confirmed findings into the issue list below. Drop anything you can positively demonstrate is a false positive, and say what you dropped and why.

6. **Residual bug scan** — cover what the delegated reviews do not, given they only see the diff:
   - Race conditions or data-integrity problems across phases
   - Edge cases implied by the plan but never exercised by any test or browser check

7. **You have access to the `neon-agent` subagent** if you need to query the database to verify schema, data integrity, or test queries.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories (the one exception is the `-history.md` file described under "Clean up cross-phase memory" below — that deletion is an expected, authorized part of this command and needs no confirmation)
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone

**Output format:**
- Test and lint results
- Phase X: [status and notes, including how it was verified]
- List any issues found, categorized as: CRITICAL (must fix), WARNING (should fix), or INFO (nice to have) — noting which came from `/code-review` or `/security-review`
- An explicit **Unverified** list: anything you could not confirm, and why
- If everything passes, confirm the implementation is complete and production-ready. Do not use that phrase if anything sits in the Unverified list — say what is left instead.

**Clean up cross-phase memory (do this last):**
- The `-history.md` file is scratch state that exists only to bridge phases during implementation. Once validation is done it has served its purpose.
- If validation passed with NO CRITICAL issues, delete the history file. Confirm in your output that you deleted it.
- If you found CRITICAL issues, do NOT delete it — the user will likely fix and re-validate, and the carried-forward notes are still useful. State that you retained the history file and why.
- Only ever delete the specific `-history.md` file derived from this plan's path. Never delete the plan itself or any other file.
