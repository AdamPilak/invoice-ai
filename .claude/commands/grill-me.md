---
description: Quiz the developer on the current change until they can explain every line
argument-hint: [optional focus, e.g. a file or topic]
---

Quiz the developer on the change on the current branch. Optional focus: $ARGUMENTS

The goal is that the developer can explain and defend every line of this change. Do not change any files.

1. **Collect the change.** Run `git diff main...HEAD` and `git diff` (uncommitted work) and read the changed files in full where context is needed. If there is no change, say so and stop.
2. **Prepare questions.** Pick 6 to 10 questions that cover, in this order of priority:
   - What a specific, non-obvious piece of code does (quote it with `file:line`).
   - Why this approach, and what the alternatives and trade-offs are.
   - What happens on failure: invalid input, network or database errors, race conditions, a retried job.
   - Security and data isolation: validation, authorization, `companyId` scoping, secrets.
   - How the change is tested, and what is not tested yet.
   - The underlying language, framework or database concepts the code relies on.
   Prefer questions about code Claude Code wrote, since the developer has seen it least.
3. **Ask one question at a time.** Wait for the answer. Then:
   - Say whether it is correct, partly correct, or wrong, and why.
   - If it is incomplete, give a short explanation or a hint and ask a follow-up.
   - Do not give away the answer before the developer tries.
4. **Use the developer's language** for the conversation (Polish or English).
5. **Finish with a summary:**
   - Topics the developer explained well.
   - Gaps to review, each with a pointer (a file, a doc link, or a concept to read about).
   - Code that should be simplified or documented because it was hard to explain.

Do not mark the task as done. That happens in `/wrap-up`.
