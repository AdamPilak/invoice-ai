---
description: Close a task - run checks, update roadmap, specs and ADRs, and prepare the commit
argument-hint: [optional task name]
---

Wrap up the current task: $ARGUMENTS

1. **Run checks.** Run the lint, typecheck and test commands listed in `CLAUDE.md` (Commands section). If any fail, report the failures and stop. Do not mark the task as done.
2. **Confirm the review steps happened.** Check this conversation for a reviewer subagent pass and, if Claude Code wrote code in this task, for `/grill-me`. If either is missing, say so and ask whether to run it now.
3. **Update `docs/ROADMAP.md`.** Mark the finished items `[x]`. If something was left out or split off, add it as a new `[ ]` item in the right place and say so.
4. **Update the spec** in `docs/specs/`. Set its status (`implemented` or `accepted`) and note any deviation from the acceptance criteria.
5. **Record decisions.** List the decisions made during the task (library choice, data model shape, trade-offs). For each one that is not trivial and not covered by an existing ADR, draft a new ADR in `docs/decisions/` using the template in `docs/WORKFLOW.md`, with the next free number.
6. **Keep docs in sync.**
   - Update `docs/ARCHITECTURE.md` if modules, flows or the data model changed.
   - Update the Commands section of `CLAUDE.md` if new scripts or tools were added.
   - Update `.env.example` if new environment variables were added.
7. **Prepare the commit.** Show `git status` and a proposed Conventional Commit message in English (`type(scope): summary` plus a short body). Commit only after the developer confirms.
8. **Report** in a few lines: what was done, what changed in the docs, and what the next roadmap item is.
