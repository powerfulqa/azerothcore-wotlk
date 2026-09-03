# Project documentation

Design record for an original, non-commercial **classless roguelite** server built on
AzerothCore 3.3.5a.

> **Not upstream AzerothCore documentation.** Upstream's own docs are in `../doc/`, and
> its agent/contributor rules are in `../AGENTS.md` and `../.agents/docs/` — those still
> apply in full.

---

## Start here

| Order | Read | Why |
|---|---|---|
| 1 | **[WORKING_AGREEMENT.md](WORKING_AGREEMENT.md)** | How work is done here: RPAC, approval gates, the non-negotiable rules. Read before touching anything. |
| 2 | **[DECISION_LOG.md](DECISION_LOG.md)** | What has been decided, and the pending-decisions table — **the next action is at the bottom of it**. |
| 3 | [PROJECT_VISION.md](PROJECT_VISION.md) | What we are building and why anyone would play it. |

## The concept in three lines

A **life** is a run from level 1 to 60. During that life you build a classless character
from curated cross-class abilities, talent-like passives and run-scoped augments. When
the life ends the build is gone; account-level progression carries forward and makes the
next life wider and faster.

## All documents

| Document | Covers |
|---|---|
| [WORKING_AGREEMENT.md](WORKING_AGREEMENT.md) | Roles, RPAC, non-negotiable rules, where things live, git and publication |
| [PROJECT_VISION.md](PROJECT_VISION.md) | Player fantasy, the "why play this" argument, audience, success metrics, legal stance |
| [DESIGN_PILLARS.md](DESIGN_PILLARS.md) | Five pillars, each an accept/reject test for any proposed feature |
| [CORE_GAME_LOOP.md](CORE_GAME_LOOP.md) | The five nested loops and the tensions designed between them |
| [BALANCE_FRAMEWORK.md](BALANCE_FRAMEWORK.md) | Balance dimensions, telemetry (M-1…M-11), anti-exploit register (X-1…X-11), hypotheses |
| [TECHNICAL_ARCHITECTURE.md](TECHNICAL_ARCHITECTURE.md) | Verified engine surface, hard client constraints, and what is explicitly **not** verified |
| [QA_STRATEGY.md](QA_STRATEGY.md) | Evidence rule, test layers, mandatory gates by change type, definition of done |
| [CONTENT_PIPELINE.md](CONTENT_PIPELINE.md) | How one ability / talent / augment goes from idea to live |
| [DECISION_LOG.md](DECISION_LOG.md) | ADRs, plus open questions ranked by cost-to-reverse |
| [requirements/project-foundation.REQUIREMENTS.md](requirements/project-foundation.REQUIREMENTS.md) | Full foundation requirements: goals, non-goals, acceptance criteria, risks, phase backlog |

## Status

**Foundation documentation only.** No code, no schema, no module, and no build has been
run. Every document is marked *Proposed* except the two accepted ADRs.

Two decisions are settled (ADR-0001 direction, ADR-0002 life-end model). Ten questions
are open and deliberately unanswered rather than guessed at.

**Next action:** answer **Q1** — is a "life" a character, or a character reset in place?
It has the highest cost to reverse of anything outstanding, and it blocks the Phase 2
schema and finalising `CORE_GAME_LOOP.md` §5.

## Conventions in these documents

`[V]` verified against this checkout, cited as `file.ext:line` · `[A]` assumption ·
`[P]` proposal awaiting approval · `[O]` open question
