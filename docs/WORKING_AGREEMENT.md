# Working Agreement

**Status:** Active. This is how work is done on this project.
**Last updated:** 2026-09-04
**Audience:** Any engineer or AI agent picking this project up.

If you are starting a session on this project, **read this file first**, then
`README.md`, then `DECISION_LOG.md` (its pending-decisions table always ends with the
next action).

---

## 1. Roles

**Chris (`powerfulqa`) is the owner and sole decision-maker.** He reviews and takes
responsibility for the code, so he must be able to understand and defend every change.

**The engineering role** combines: AzerothCore/C++ engineer, systems and database
engineer, technical game designer, live-service and balance analyst, QA lead, and
documentation/release engineer.

**Never make a major product, architectural, balance, schema or gameplay decision
alone.** Present options with trade-offs and wait for approval. Routine judgement calls
are yours to make; costly-to-reverse ones are not.

---

## 2. RPAC

Every feature, bug, system or balance change follows four phases.

### REFINE
- Inspect the existing material first — code, schema, conventions, config, tests.
- Ask **one focused question at a time**. Not a batch.
- Write `.claude/plans/<slug>/<slug>.REQUIREMENTS.md`: goals, non-goals, player-facing
  behaviour, technical requirements, acceptance criteria, balance hypotheses,
  telemetry, risks, open questions, test approach.
- **Stop. Ask for approval.**

### PLAN
- Read the approved requirements. Investigate the specific code paths and data.
- Write `.claude/plans/<slug>/<slug>.PLAN.md`: precise files, functions, tables,
  migrations, implementation order, test cases, rollback, performance considerations,
  manual in-game test steps.
- **Separate facts verified from the repository from assumptions.** Explicitly.
- **Stop. Ask for approval.**

### ACT
- Execute only the approved plan, in small logical changes.
- Run formatting, compilation, tests, static checks and database validation.
- Report exactly what changed and the **real** outputs.
- If an unexpected design choice appears, stop and ask rather than guessing.

### CONSOLIDATE
- Review the diff as a hostile reviewer: regressions, exploits, security, performance,
  balance, missing tests, schema mistakes, unclear player experience.
- Produce a concise self-review report stating what was verified **and what was not**.
- Update docs, tests, release notes, configuration and migration/rollback notes.
- Ask the owner to review the diff and confirm in-game testing.

Work in focused phases. Recommend `/clear` before a new phase when context would
otherwise be cluttered.

---

## 3. Non-negotiable rules

1. The owner decides. Options and approval, never unilateral major decisions.
2. Follow RPAC. Never jump from an idea straight into broad implementation.
3. **Do not "vibe code".**
4. Inspect the codebase, AzerothCore conventions, schema, module structure, config and
   tests **before** modifying anything.
5. **Do not invent APIs, database fields, hooks, commands or core behaviour.** Verify
   them in the local source or official AzerothCore documentation first, and cite
   `file.ext:line`.
6. Production-quality, maintainable, narrow-scope code matching repository conventions.
7. Explain important technical choices in plain English.
8. Small, reviewable commits. No unrelated formatting changes, no drive-by refactors.
9. **"It compiles" is insufficient.** Every change needs appropriate automated checks
   and practical in-game verification.
10. **Never claim a build, test, migration or in-game check passed unless it was
    actually run and its result shown.**
11. Call out uncertainty, assumptions, risks, missing information, exploits,
    performance impacts, compatibility concerns and data-migration risks early.
12. Keep all work original and non-commercial. No Blizzard assets, client files, game
    data, music, text or branding beyond what is legally appropriate for an
    independently operated development environment.
13. Prefer custom modules. Keep core patches to an absolute minimum and document every
    unavoidable divergence as an ADR.

---

## 4. Responding to a new idea

Do not start implementation. Respond first with:

1. What you understand.
2. Assumptions you are making.
3. The single most important decision or question needing an answer.
4. The suggested next artefact — normally a REQUIREMENTS document.

Then ask that one question and wait.

---

## 5. Where things live

| Path | What | Tracked? |
|---|---|---|
| `docs/` | Published design record — the durable source of truth | Yes |
| `docs/requirements/` | Promoted requirements documents | Yes |
| `.claude/plans/<slug>/` | Scratch for in-progress RPAC artefacts | **No — gitignored** |
| `AGENTS.md`, `.agents/docs/` | Upstream AzerothCore agent rules; still fully apply | Yes (upstream) |

**Promotion workflow:** draft requirements and plans in `.claude/plans/<slug>/`. Once
the owner approves one, promote a copy into `docs/` so it survives a fresh clone and is
visible to reviewers. Scratch stays scratch.

**Note:** `AGENTS.md` mandates `.agents/plans/` for planning docs while the owner's
standing instruction mandates `.claude/plans/`. Both are gitignored. We follow the
owner's instruction; the divergence is tracked as open question Q10.

---

## 6. Git and publication

- `origin` is upstream `azerothcore/azerothcore-wotlk`. **Never push to it.**
- `fork` is `powerfulqa/azerothcore-wotlk` (public). All project work goes there.
- When opening a PR, **always** pass `--repo powerfulqa/azerothcore-wotlk`.
  `gh pr create` inside a fork otherwise defaults its base to the upstream project,
  which would open a public PR against AzerothCore itself.
- Commit style follows the repository's convention: `type(Scope): short subject`
  (see `.git_commit_template.txt`).
- Commit or push only when the owner asks.

---

## 7. Standards that live elsewhere

Do not duplicate these here — read them:

| Concern | Document |
|---|---|
| What we are building and why | `PROJECT_VISION.md` |
| Accept/reject test for any feature | `DESIGN_PILLARS.md` |
| The loops the game is made of | `CORE_GAME_LOOP.md` |
| Balance dimensions, telemetry, anti-exploit register | `BALANCE_FRAMEWORK.md` |
| Verified engine surface, constraints, what is unverified | `TECHNICAL_ARCHITECTURE.md` |
| Test layers, mandatory gates, definition of done | `QA_STRATEGY.md` |
| How content goes from idea to live | `CONTENT_PIPELINE.md` |
| Decisions taken, and what is still open | `DECISION_LOG.md` |
| Full foundation requirements | `requirements/project-foundation.REQUIREMENTS.md` |
| AzerothCore build, C++, SQL, e2e, review rules | `../AGENTS.md` → `../.agents/docs/` |
