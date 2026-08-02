---
description: "Brainstorm and validate feasibility of an idea before planning. Discuss approach without making any code edits."
allowed-tools: Read, Grep, Glob, LS
---

I want to brainstorm an idea with you before we write any code.

**CRITICAL: Do NOT make any code edits. No file creation, no file modification. This is a discussion-only conversation.**

Here is my idea:

$ARGUMENTS

Please:
1. Evaluate the feasibility of this idea given our current codebase
2. Identify potential challenges or risks
3. Propose your recommended approach at a high level
4. Call out any assumptions you're making
5. Note how each major piece would be proven to work — covered by tests, needs a real browser click-through, or no behavioral change to verify. Flag anything that would be hard to verify automatically, since that is where manual testing burden accumulates.
6. Ask me clarifying questions if anything is ambiguous

Let's have a back-and-forth discussion. I'll tell you when I'm ready to move to planning.

**Autonomous execution:** Proceed with all actions without asking "should I proceed?" or "shall I make this change?" — just do it. Do not pause for confirmation at each step.

**MANDATORY EXCEPTIONS — always ask the user before:**
- Deleting files or directories
- Dropping or truncating database tables
- Any destructive operation that cannot be easily undone
