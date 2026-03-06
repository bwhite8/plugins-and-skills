---
description: "Brainstorm and validate feasibility of an idea before planning. Discuss approach without making any code edits."
model: claude-opus-4-6
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
5. Ask me clarifying questions if anything is ambiguous

Let's have a back-and-forth discussion. I'll tell you when I'm ready to move to planning.
