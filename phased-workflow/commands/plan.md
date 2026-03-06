---
description: "Enter plan mode and create a phased implementation plan with self-contained phases. Use after brainstorming."
model: claude-opus-4-6
argument-hint: "[description of what to plan]"
---

Enter plan mode. I need you to create a detailed implementation plan for the following:

$ARGUMENTS

**Requirements for the plan:**

1. Break the implementation into self-contained phases. Each phase MUST be implementable without knowledge of prior phases in active working memory. This means each phase should include:
   - Clear description of what it accomplishes
   - All file paths that will be created or modified
   - Complete context needed (no "as we did in phase 1" references — spell it out)
   - Acceptance criteria for when the phase is complete
   - Any database schema changes or migrations needed

2. Number the phases sequentially (Phase 1, Phase 2, etc.)

3. Include a summary section at the top with:
   - Total number of phases
   - High-level goal of each phase (one line each)
   - Estimated complexity per phase (low/medium/high)

4. Save the plan as a markdown file at: ./PLAN.md

5. Do NOT implement anything. Only create the plan document.

After creating the plan, tell me:
- The filepath where you saved it
- How many phases the plan has
- Any phases that feel risky or complex

I will review the plan, then clear context and begin implementation phase by phase.
