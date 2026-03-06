---
description: "Validate that a completed plan was fully and accurately implemented with no critical bugs."
model: claude-opus-4-6
argument-hint: "[path/to/PLAN.md]"
---

Read the implementation plan at: $ARGUMENTS

This plan has been fully implemented across all phases. Your job is to validate the implementation.

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

**Output format:**
- Phase X: [status and notes]
- List any issues found, categorized as: CRITICAL (must fix), WARNING (should fix), or INFO (nice to have)
- If everything passes, confirm the implementation is complete and production-ready.
