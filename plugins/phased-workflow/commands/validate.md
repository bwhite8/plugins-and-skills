---
description: "Validate that a completed plan was fully and accurately implemented with no critical bugs."
argument-hint: "[path/to/PLAN.md]"
---

Read the implementation plan at: $ARGUMENTS

This plan has been fully implemented across all phases. Your job is to validate the implementation.

**Load cross-phase memory:**
- Derive the history file path from the plan path "$ARGUMENTS" by replacing the trailing `.md` with `-history.md` (e.g. `.../add-user-auth.md` → `.../add-user-auth-history.md`).
- If that file exists, read it. It holds the notes each phase carried forward — decisions, deviations from the plan, and gotchas. Use it to focus your validation: deviations and gotchas recorded there are prime spots to scrutinize.

**Perform the following validation:**

1. **Completeness check**: Go through every phase and verify that each acceptance criterion has been met. List any gaps.

2. **Integration check**: Verify that all phases work together correctly. Look for:
   - Missing imports or broken references between phases
   - Inconsistent naming or interfaces
   - Database schema mismatches

3. **Critical bug scan**: Review the implemented code for:
   - Unhandled errors or edge cases
   - Security issues (SQL injection, auth gaps, exposed secrets)
   - Race conditions or data integrity problems
   - Missing error handling on API calls or DB queries

4. **You have access to the `neon-db` subagent** if you need to query the database to verify schema, data integrity, or test queries.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories (the one exception is the `-history.md` file described under "Clean up cross-phase memory" below — that deletion is an expected, authorized part of this command and needs no confirmation)
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone

**Output format:**
- Phase X: [status and notes]
- List any issues found, categorized as: CRITICAL (must fix), WARNING (should fix), or INFO (nice to have)
- If everything passes, confirm the implementation is complete and production-ready.

**Clean up cross-phase memory (do this last):**
- The `-history.md` file is scratch state that exists only to bridge phases during implementation. Once validation is done it has served its purpose.
- If validation passed with NO CRITICAL issues, delete the history file. Confirm in your output that you deleted it.
- If you found CRITICAL issues, do NOT delete it — the user will likely fix and re-validate, and the carried-forward notes are still useful. State that you retained the history file and why.
- Only ever delete the specific `-history.md` file derived from this plan's path. Never delete the plan itself or any other file.
