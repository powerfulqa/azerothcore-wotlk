# Decision Log

**Status:** Living document.
**Last updated:** 2026-09-03
**Purpose:** Architecture Decision Records for decisions that are costly to reverse. A decision that is not
here was not made — it was assumed, and assumptions get challenged.

**Record states:** `Accepted` · `Proposed` (awaiting owner approval) · `Superseded` · `Rejected`

---

## Index

| ADR | Title | State | Date |
|---|---|---|---|
| 0000 | Direction: persistent classless world without a run layer | **Superseded** by 0001 | 2026-09-03 |
| 0001 | Direction: persistent-progression roguelite with per-life classless builds | **Accepted** | 2026-09-03 |
| 0002 | Life-end model: voluntary prestige plus opt-in stakes | **Accepted** | 2026-09-03 |
| 0003 | Adopt azerothMCP as a Phase-1 verification tool | Proposed | 2026-09-03 |
| 0004 | Project documentation lives in `docs/` | Proposed | 2026-09-03 |
| 0005 | Design documentation is published publicly on the fork | **Accepted** | 2026-09-04 |
| 0006 | `CLAUDE.md` carries a project pointer (accepted upstream divergence) | **Accepted** | 2026-09-04 |

---

## ADR-0000 — Direction: persistent classless world without a run layer

**State:** Superseded by ADR-0001 · **Date:** 2026-09-03

**Context.** During initial refinement, the direction was recorded as a persistent classless world with *no*
run resets, explicitly excluding a hub, meta-currency and any ephemeral power layer, with the instruction
that these should not leak into the schema.

**Why it was superseded.** The subsequent project brief reinstated the run loop: a life is a level 1→60 run,
run-scoped augments reset, and account-scoped progression persists. The exclusions in this record were
therefore wrong.

**Consequence.** Recorded rather than deleted, because it is the reason `project-foundation.REQUIREMENTS.md`
has a v2 and a superseded v1. Superseding this early cost nothing; discovering it after a schema existed
would have been expensive.

---

## ADR-0001 — Direction: persistent-progression roguelite with per-life classless builds

**State:** Accepted (owner) · **Date:** 2026-09-03 · **Supersedes:** ADR-0000

**Decision.** A **life** is a level 1→60 run. Within a life the player constructs a classless build from
curated cross-class abilities, talent-like passives and roguelite augments. Run-scoped progress resets at
life end; account-scoped progression persists and compounds, favouring **agency before raw power**.

**Consequences.**
- The classic 1–60 world is the delivery vehicle, not scenery (Pillar 1).
- Three power layers exist and are never conflated: abilities, talents, augments.
- Persistent state has no reset valve, so schema quality matters from the first table (T-6).
- Distinctiveness must come from the systems, since the content is stock — this is Risk R-1, and
  `PROJECT_VISION.md` §3 exists to keep it under scrutiny.

**Cost to reverse.** Very high once persistent schema exists. This is the decision everything else hangs from.

---

## ADR-0002 — Life-end model: voluntary prestige plus opt-in stakes

**State:** Accepted (owner) · **Date:** 2026-09-03

**Context.** The brief specifies both a level-60 endgame the player lives in (3-player group baseline,
repeated high-mastery solo raid attempts) *and* a run loop that resets to level 1. These compete for the
same character. Three models were considered: voluntary prestige only; death ends the life; hybrid.

**Decision.** **Level 60 is a home, not a finish line.**
- The default life ends only when the player *chooses* to prestige.
- Optional **stake modifiers** (hardcore, challenge conditions) are declared at life start, visible for the
  life's duration, and immutable once chosen. They grant additional persistent reward proportionate to risk.

**Rationale.** A death-ends-life default directly contradicts both the 3-player raid endgame and repeated
solo raid attempts, since both require surviving failure. Voluntary prestige alone leaves the roguelite loop
with no tension and risks players parking at 60 permanently. The hybrid satisfies every stated constraint —
solo-friendly first life, a real endgame, and genuine tension for those who want it.

**Consequences.**
- The life record carries a **ruleset/modifier field** from day one.
- **Two death paths** to design, implement, test and balance.
- **D2-a** No persistent unlock may be reachable *only* through a staked life, or "optional" is false.
- **D2-b** Staked life endings must be fully auditable — cause, location, source, timestamp. A disputed
  hardcore death is a support incident.
- **D2-c** Disconnect and server-fault deaths need a stated policy (open: Q9).

**Cost to reverse.** High. The modifier field and audit trail are schema-level; adding them later means
migrating live accounts.

---

## ADR-0003 — Adopt azerothMCP as a Phase-1 verification tool

**State:** Proposed · **Date:** 2026-09-03

**Context.** Three of the four unverified items blocking design (V-1 encounter tuning, V-2 cross-chassis
spell behaviour, V-3 quest path coverage) are database-shaped questions.
`https://github.com/azerothcore/azerothMCP` is an official AzerothCore-org, GPL-2.0, actively maintained
read-only MCP server over the world/characters/auth databases (verified via `gh api`, last push 2026-04-15).

**Decision (proposed).** Adopt it as a verification tool, installed **immediately before Phase 1** — not
earlier, because it reads a populated MySQL and this project has no build and no database yet.

**Constraints.** Read-only inspection only; never a write path — schema change continues through
`pending_db_*/` (T-5). It does not relax T-2: facts it reports are cited with the query that produced them.
`ENABLE_SOURCE_CODE` and `ENABLE_WIKI` stay off unless specifically needed.

**Cost to reverse.** Low — it is tooling, not architecture.

---

## ADR-0004 — Project documentation lives in `docs/`

**State:** Proposed · **Date:** 2026-09-03

**Context.** `.claude/plans/**` is gitignored (`.gitignore:68`), so planning documents are not
version-controlled. Design decisions need a durable, reviewable home.

**Decision (proposed).** `docs/` holds the durable design record and is version-controlled.
`.claude/plans/<slug>/` remains working scratch for RPAC artefacts.

**Known wrinkles.**
- Upstream AzerothCore already ships `doc/` (`changelog`, `ConfigPolicy.md`, `Logging.md`). Our `docs/` sits
  beside it. The adjacency is slightly confusing; a rename to something like `docs/design/` is available if
  preferred.
- `[O]` **Q10** remains open: `AGENTS.md` mandates planning docs in `.agents/plans/`, while the owner's
  standing instruction mandates `.claude/plans/`. Both are gitignored. Currently following the owner's
  instruction and flagging the divergence rather than silently choosing.

**Cost to reverse.** Low — a directory move.

---

## ADR-0005 — Design documentation is published publicly on the fork

**State:** Accepted (owner) · **Date:** 2026-09-04

**Context.** The design documents needed a review venue so a third party could comment on
them. Options considered: a private docs-only repository; a public fork of AzerothCore
with a PR; a private full mirror of the repository.

**Decision.** Fork `azerothcore/azerothcore-wotlk` to `powerfulqa/azerothcore-wotlk`
(public — a fork of a public repository cannot be made private) and review via a PR
against **the fork's own master**. Design docs and any future module code stay in one
place, and pushing costs almost nothing because the history is shared.

**Consequences.**
- **Everything committed to the fork is public and permanent.** Anything that should not
  be published needs a separate private repository, decided before it is committed.
- `origin` remains upstream and is never pushed to. The fork is a second remote, `fork`.
- `gh pr create` inside a fork defaults its base to the **parent** repository, so every
  PR must pass `--repo powerfulqa/azerothcore-wotlk`. Without it, a PR would be opened
  publicly against the AzerothCore project itself.
- Identity exposure was reviewed and accepted: the owner's handle is public; his email
  address is not, and does not appear in any commit (commits use the GitHub
  `users.noreply.github.com` address).

**Cost to reverse.** High for anything already pushed — publication is effectively
permanent regardless of later deletion.

---

## ADR-0006 — `CLAUDE.md` carries a project pointer (accepted upstream divergence)

**State:** Accepted (owner) · **Date:** 2026-09-04

**Context.** A fresh engineer or agent loads `CLAUDE.md` → `AGENTS.md`, which is
upstream AzerothCore's generic contributor guidance. Nothing in it referenced this
project's documents or working method, so a new session had no route to `docs/` and
would plausibly begin implementing without an approved plan.

**Decision.** Add a short pointer section to `CLAUDE.md` directing the reader to
`docs/WORKING_AGREEMENT.md`, `docs/README.md` and `docs/DECISION_LOG.md`.

**Consequences.**
- `CLAUDE.md` is an upstream-tracked file, so this is a **permanent, deliberate
  divergence** that will conflict on every rebase touching that file. It is small and
  trivially re-applied.
- The rejected alternative was `CLAUDE.local.md` (gitignored, zero conflict), which was
  declined because it does not survive a fresh clone — and surviving a fresh clone is the
  entire point.
- This is the project's first accepted core-tree divergence. Per rule 13 in the working
  agreement, every further one needs its own ADR.

**Cost to reverse.** Trivial — delete the section.

---
## Pending decisions

Open questions, in the order they will be asked. Each becomes an ADR when answered.
Full text: `requirements/project-foundation.REQUIREMENTS.md` §14.

| Q | Decision | Blocks | Cost to reverse |
|---|---|---|---|
| **Q1** | Is a life a character, or a character reset in place? | Phase 2 schema, persistence anchor, `CORE_GAME_LOOP.md` §5 | **Very high** |
| Q2 | Acquisition model: random roll / choice-of-three / protected categories / rerolls / hybrid | Phase 4, and the feel of every life | High |
| Q3 | Power source: existing spells cross-chassis / original effects / mix | Phase 1 scope; curation vs C++ effects project | High |
| Q4 | Module hosting: separate repo vs vendored in-tree | Phase 1 setup | Medium |
| Q5 | Audience scale and realm openness | Anti-exploit budget, telemetry investment | Medium |
| Q6 | Chassis visibility to the player | Player-facing framing | Low |
| Q7 | Nerf policy for already-acquired powers | X-11 enforcement credibility | Medium |
| Q8 | Persistent currency shape | Phase 2/3 schema | High |
| Q9 | Disconnect / server-fault deaths in staked lives | Player trust; D2-c | Medium |
| Q10 | Planning-doc location vs `AGENTS.md` | Process only | Low |

**Next:** Q1.
