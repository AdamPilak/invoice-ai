# Workflow

How work moves from an idea to a commit in this project. The tools are Claude Code (implementation) and project chats in Claude (specs and reviews).

## Principles

- The repository is the source of truth. Decisions made in a chat are not real until they are written to `docs/`.
- One feature or stage per chat. One task per Claude Code session.
- The developer writes the core areas listed in `CLAUDE.md` (AI pairing rules); Claude Code writes the rest.
- The developer must be able to explain every line of code in the repository.

## Who writes the code

| Area | Who writes the code | What Claude does |
| --- | --- | --- |
| Core areas (see `CLAUDE.md`) | The developer | Hints, guiding questions, docs links, review. No solution code unless explicitly asked. |
| Everything else | Claude Code | Writes small, reviewable diffs, explains non-obvious choices, lists what to verify. The developer reviews every line. `/grill-me` is mandatory. |

## The cycle

1. **Spec (chat).** Agree on scope, acceptance criteria, edge cases and open questions. Save the spec as `docs/specs/<feature>.md` using the template below.
2. **Plan (Claude Code, plan mode).** Start a fresh session. Claude reads `CLAUDE.md`, the spec and relevant ADRs, then proposes a plan. Adjust the plan before any code is written.
3. **Implement.** Core areas: the developer writes, Claude assists. Everything else: Claude writes in small steps.
4. **Test.** Add or update Vitest, RTL and Playwright tests as appropriate. Lint and typecheck must pass.
5. **Subagent review.** Run a review subagent on the diff (correctness, security, tenant isolation, missing tests).
6. **/grill-me.** Claude quizzes the developer on the change: why this approach, alternatives, failure modes. Mandatory when Claude Code wrote the code, recommended otherwise.
7. **/wrap-up.** Update `docs/ROADMAP.md` (task status) and add or update ADRs in `docs/decisions/`.
8. **Commit.** Conventional Commits in English.
9. **Review (chat).** A question-driven review of the finished feature: why this design, what the alternatives were, what breaks at scale, how it is tested.

## Spec template

```markdown
# <Feature name>

Status: draft | accepted | implemented
Roadmap: <version / week>

## Goal
One or two sentences.

## User stories
- As a <role>, I want <action> so that <benefit>.

## Acceptance criteria
- [ ] ...

## Out of scope
- ...

## API / data changes
Endpoints, schemas, migrations.

## Edge cases
- ...

## Open questions
- ...
```

## ADR template

Files in `docs/decisions/` are numbered: `NNN-short-title.md`.

```markdown
# NNN. <Title>

Date: YYYY-MM-DD
Status: proposed | accepted | superseded by NNN

## Context
## Decision
## Alternatives considered
## Consequences
```

## Commits and branches

- Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`, `ci:`.
- Optional scope with the app or feature: `feat(api/documents): add presigned upload endpoint`.
- Short-lived branches per task (`feat/document-upload`), merged through PRs once CI is green.

## Language

- Chats: Polish.
- Repository (code, commits, PRs, docs): English.
- App UI: Polish in 1.0; English switch via i18n from 1.1.
