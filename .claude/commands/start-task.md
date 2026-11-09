---
description: Start a roadmap task - load context, check the spec, decide who writes the code, and propose a plan
argument-hint: <roadmap task or short description>
---

Start work on this task: $ARGUMENTS

Do not write or change any code in this step. The output is a plan for the developer to approve.

1. **Load context.** Read `CLAUDE.md`, `docs/ROADMAP.md`, `docs/ARCHITECTURE.md`, and every ADR in `docs/decisions/` that relates to the task.
2. **Find the task in the roadmap.** Quote the matching item(s) and the week they belong to. If the task is not in the roadmap, or belongs to version 1.1+ while 1.0 is in progress, say so and stop until the developer confirms.
3. **Check the spec.** Look for the relevant file in `docs/specs/`. If there is none, or it lacks acceptance criteria, stop and say what is missing. Do not invent requirements.
4. **Decide who writes the code.** Check the task against the AI pairing rules in `CLAUDE.md`:
   - If it falls in a developer-owned area, say so clearly. The plan is then a guide for the developer: steps, concepts to understand, docs to read, and questions to think about. No implementation code.
   - Otherwise, Claude Code will write the code. The plan lists the changes in small, reviewable steps.
   - If the task is split between both, mark which steps belong to whom.
5. **Inspect the current code** the task will touch, so the plan matches what exists.
6. **Propose the plan**, containing:
   - Goal in one sentence and the acceptance criteria from the spec.
   - Files to create or change, in order.
   - Data model, API or schema changes, if any.
   - Tests to add (unit, component, e2e).
   - Risks and open questions.
   - A branch name following `docs/WORKFLOW.md` (for example `feat/document-upload`).
7. **Wait** for the developer to approve or adjust the plan before doing anything else.

After approval, create the branch and mark the task as in progress (`[~]`) in `docs/ROADMAP.md`.
