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
| [REFERENCES.md](REFERENCES.md) | The AzerothCore wiki and other official sources — where they live, and how much weight each carries |
| [requirements/project-foundation.REQUIREMENTS.md](requirements/project-foundation.REQUIREMENTS.md) | Full foundation requirements: goals, non-goals, acceptance criteria, risks, phase backlog |

## Status

**Foundation documentation only.** No code, no schema, no module, and no build has been
run. Every design document is marked *Proposed* except the accepted ADRs.

Sixteen decisions are settled (one superseded the same day):

| ADR | In one line |
|---|---|
| 0001 | A life is a level 1→60 run; classless build per life; account progression persists |
| 0002 | Level 60 is a home, not a finish line — voluntary prestige plus opt-in stakes |
| 0005 | Design docs are published publicly on the fork |
| 0006 | `CLAUDE.md` carries a project pointer (first accepted upstream divergence) |
| **0007** | The character is a persistent **vessel**, reset in place at prestige; each life is a first-class row |
| **0008** | Acquisition is a choice-of-N draft; protected categories guarantee a function is *offered*, never forced; rerolls are earned |
| **0009** | A wild card start, then at every level a choice of new ability, **upgrade**, or talent |
| **0010** | Abilities and upgrades are curated client spells; talents and augments are original server-authored effects |
| ~~0011~~ | *Superseded by 0012 the same day* |
| **0012** | **Every chassis carries every resource** — mana, rage and energy at once, so any chassis may be offered any ability |
| **0013** | A **bundled, required client AddOn** ships the stacked multi-resource frame — **amends T-10** |
| 0014 | *Reserved — held in the private decision store (ADR-0005)* |
| **0015** | The server runs in **Docker on a dedicated remote host** — upstream's own compose stack, no divergence |
| **0016** | The private decision store is a **local repository with an encrypted offline backup** — no third party holds it |
| **0017** | Live e2e runs against a **full local Docker stack**; the production host is never a test target |
| **0018** | Backups are **encrypted logical dumps of `auth`+`characters`** with an automated restore test; `world` is rebuilt from the repo |

Those answers raised ten further questions (Q11–Q20), one of which (Q18) was dissolved
rather than answered. Fourteen remain open and are deliberately unanswered rather than
guessed at.

**Next action:** **Q11** — may prestige change the vessel's chassis? Re-read its
cost-to-reverse first: ADR-0012 gave every chassis every resource, which defused most of
what made this urgent. Then **Q24**.

## Conventions in these documents

`[V]` verified against this checkout, cited as `file.ext:line` · `[A]` assumption ·
`[P]` proposal awaiting approval · `[O]` open question
