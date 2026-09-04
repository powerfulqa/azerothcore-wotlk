# Decision Log

**Status:** Living document.
**Last updated:** 2026-09-04
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
| 0007 | Life model: persistent character vessel with first-class life records | **Accepted** | 2026-09-04 |
| 0008 | Acquisition model: guaranteed-offer draft with an earned reroll budget | **Accepted** | 2026-09-04 |
| 0009 | Acquisition cadence: a per-level choice between breadth, depth and passives | **Accepted** | 2026-09-04 |
| 0010 | Power source: curated client spells for what the player sees, original effects for what warps it | **Accepted** | 2026-09-04 |
| 0011 | Cross-chassis resource policy: chassis keep their native resource, abilities curated to match | **Superseded** by 0012 | 2026-09-04 |
| 0012 | Every chassis carries every resource | **Accepted** | 2026-09-04 |
| 0013 | A bundled client AddOn ships the multi-resource frame (amends T-10) | **Accepted** | 2026-09-04 |
| 0014 | *Reserved — held in the private decision store (ADR-0005)* | **Accepted** | 2026-09-04 |
| 0015 | The server runs in Docker on a dedicated remote host | **Accepted** | 2026-09-04 |
| 0016 | The private decision store is a local repository with an encrypted offline backup | **Accepted** | 2026-09-05 |
| 0017 | Live e2e runs against a full local Docker stack on the development machine | **Accepted** | 2026-09-05 |
| 0018 | Backups are encrypted logical dumps of the irreplaceable databases, with an automated restore test | **Accepted** | 2026-09-05 |

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
## ADR-0007 — Life model: persistent character vessel with first-class life records

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Answers:** Q1

**Context.** ADR-0001 defines a life as a level 1→60 run and ADR-0002 makes voluntary prestige the default
ending, but neither said what a life *is* in the database. Three models were considered: **(A)** each life is
a new character, retired at prestige; **(B)** the same character is reset in place, with no life record;
**(C)** the character is a persistent vessel reset in place, with each life recorded as a first-class row.

`[V]` Option A is structurally bounded by the client. `CharactersPerRealm` is documented as `Range: 1-10`,
`Default: 10 - (Client limitation)` (`src/server/apps/worldserver/worldserver.conf.dist:2031-2036`) and the
config validator rejects any value above 10 (`src/server/game/World/WorldConfig.cpp:230`). One character slot
per life therefore caps a player at ten lives unless character rows are archived out of `characters` — which
reaches into every guid-keyed table.

**Decision.** **C.** The character is a persistent **vessel**; prestige resets that vessel in place to level 1.
Each life is a first-class row in a project-owned table (working name `character_life`) carrying its stake
modifiers, start and end timestamps, outcome and audit trail.

**Rationale.** ADR-0002 already requires somewhere to hold the immutable stake modifiers and D2-b's auditable
life endings. Once that row exists, option B's loss of history disappears, and option A's collision with the
client slot cap is a cost paid for a benefit C already provides.

**Consequences.**

- **Three persistence anchors now exist and must never be conflated:** *account*, *vessel* (character guid),
  and *life*. Which anchor each persistent unlock hangs from is a per-feature decision and is **not** settled
  here — it interacts with Q8.
- The 10-slot cap no longer bounds lives. It bounds **concurrent vessels**: a player may keep several vessels
  running under different stake modifiers.
- **D7-a — the prestige reset is the highest-risk operation in the project.** It mutates a live character row
  and its dependants, with no undo. `[V]` `data/sql/base/db_characters/` ships 108 table files, 35 named
  `character_*`, and 51 declaring a `guid int unsigned` column — though not all of those mean *character*
  guid. **Before any implementation, an enumerated table-by-table reset manifest is required** — every table
  classified *cleared* / *preserved* / *archived to the life record* — derived from the schema in this
  checkout, not from memory. This becomes a mandatory gate in `QA_STRATEGY.md`.
- **D7-b — new failure mode: the partially-applied reset.** The reset must be atomic or resumable, and a
  vessel caught mid-reset must be unambiguously detectable. A half-reset vessel is both a corruption and an
  exploit vector.
- Life history becomes queryable, so a player-facing "past lives" view is possible. Not committed to here.
- **D7-c** Deleting a vessel must not orphan or destroy life records that back account-level unlocks. Policy
  is open — Q12.
- `[O]` `CORE_GAME_LOOP.md` §5.1 has the player select a chassis at life start. Under a persistent vessel the
  chassis is fixed for the vessel's entire existence unless prestige may change it. That is a new question —
  **Q11** — and it bears directly on how classless the game feels *across* lives, not just within one.

**Cost to reverse.** Very high once persistent schema exists. This is the second load-bearing decision after
ADR-0001.

---

## ADR-0008 — Acquisition model: guaranteed-offer draft with an earned reroll budget

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Answers:** Q2

**Context.** P-3 left the acquisition model open as the largest design question, and Pillar 2 names it as the
one that every other system inherits. Of the five candidates in `requirements §14`, only two are models —
random grant and choice-of-N draft; "protected categories" and "rerolls" are modifiers on either.

`[V]` Two engine facts were checked first, because they bound the presentation under T-10:
- A gossip menu holds up to 32 options — `GOSSIP_MAX_MENU_ITEMS 32`
  (`src/server/game/Entities/Creature/GossipDef.h:30`), ASSERT-enforced
  (`src/server/game/Entities/Creature/GossipDef.cpp:43`). **N is a design choice, not a technical ceiling.**
- `gossip_menu_option.BoxText` renders a Yes/No confirmation window with arbitrary body text (wiki:
  [gossip_menu_option](https://www.azerothcore.org/wiki/gossip_menu_option)). This is the Pillar 4 legibility
  affordance on a stock client, and the same mechanism P-11 needs for prestige confirmation.

**Decision.** A **choice-of-N draft**, with two structural additions:

1. **Protected categories guarantee that a role-critical function is _offered_, not _possessed_.** By defined
   levels the player will have been *offered* healing, a defensive, an AoE and so on. They remain free to
   decline. This removes the dead-build failure mode (X-1, X-2, P-5) **without** taking the decision away,
   so Pillar 2 stays intact and lives still diverge.
2. **An earned reroll budget.** Rerolls are unlocked account-side and spent within a life — Pillar 3's
   canonical "widen the choice before raising the number".

**N starts at 3 and is a tuning value, not an architectural one.** Per Pillar 5 it must be changeable on
evidence, and Pillar 3 explicitly endorses "+1 draft option" as a persistent unlock, so N varies by player.
**Consequence: offers are stored as rows, never as three fixed columns.** A schema that hardcodes three
options cannot express its own reward economy.

**Rationale.** Pure random collides with Pillar 2's Reject column verbatim ("a grant with no alternative
forgone") and starves Pillar 3 — with no choices to widen, the persistent economy can only sell pool width
and raw power, the exact failure Pillar 3 exists to prevent. A bare draft passes Pillar 2 but leaves P-5 role
coverage to luck. The guarantee is what converts "usually fine" into "cannot be broken", and framing it as
*offer* rather than *possession* is what stops it flattening every life into the same shape.

**Consequences.**

- **D8-a — the offer is generated once and persisted, transactionally with the acquisition point.** If the
  offer is recomputed when the gossip menu opens, relogging is a free reroll. This is X-8-shaped and is
  registered as **X-12**. It is far cheaper to prevent in the schema now than to detect later.
- **D8-b — pending-offer state is life-scoped.** ADR-0007's life row is its natural home, as is the reroll
  balance. A player who logs out mid-draft is an ordinary case to be resumed, not an edge case.
- **D8-c — reroll budget spans two anchors:** granted account-side, spent life-side. Per ADR-0007's
  three-anchor rule this split is deliberate and must be explicit in the schema.
- **D8-d — P-4's tag system needs a distinct role-function axis.** Protected categories cannot be expressed
  with general-purpose tags; the taxonomy is a first-class dimension, not a tag convention.
- Every acquisition writes to the acquisition ledger already required by M-2, M-3, M-4 and M-10 — including
  **the options not taken**, without which pick-rate metrics measure availability rather than preference.
- `[O]` Two questions this raises: **Q13** (the protected-category taxonomy itself) and **Q14** (decline
  semantics — may a draft be refused outright, and do unpicked powers return to the pool later in the life?).

**Cost to reverse.** High. The offer ledger, reroll accounting and role-function taxonomy are schema-level and
Phase 4 is built on top of them. The *value* of N is trivially reversible by design.

---

## ADR-0009 — Acquisition cadence: a per-level choice between breadth, depth and passives

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Extends:** ADR-0008

**Context.** ADR-0008 settled the *shape* of acquisition (a guaranteed-offer draft with rerolls) but left its
*cadence* unstated — how often a choice occurs, and what sits on the menu.

**Decision (owner, verbatim intent).**
- A life opens with a **wild card** starting ability or talent selection.
- **At every level** the player chooses one of three things: a **new ability**, an **upgrade to an ability
  they already hold**, or a **talent**.

**Consequences.**

- **Cadence is per level** — 59 choice points across a 1→60 life. Against P-8's ~6h informed first life that
  is roughly one decision every six minutes, which is a plausible roguelite rhythm but is now a *pacing
  commitment*, not an incidental detail.
- **D9-a — the upgrade is a fourth axis.** ADR-0001 names three power layers (abilities, talents, augments);
  an upgrade is none of them. It is *depth on an existing ability*, and it competes for the same slot as
  breadth. This strengthens Pillar 2 — "go wider or go deeper" is a decision with a real cost — but the
  three-layer language in `PROJECT_VISION.md`, `CORE_GAME_LOOP.md` §5.2 and P-3/P-6 now needs an explicit
  fourth term, or the model will be described wrongly in every later document.
- `[V]` **D9-b — upgrades are expressible on a stock client with no patch.** `spell_ranks` is a world DB
  table (`data/sql/base/db_world/spell_ranks.sql:23`) with chain resolution in `SpellMgr`
  (`GetFirstSpellInChain`, `GetNextSpellInChain`, `GetSpellRank` — `src/server/game/Spells/SpellMgr.h:675-679`).
  Each rank is a distinct client spell carrying its own name, icon and tooltip, so an upgrade is legible
  under T-10 for free. Chains are data, so we are not restricted to Blizzard's own rank ladders.
- **D9-c — protected categories must account for upgrades.** Under ADR-0008 a guarantee ensures a
  role-critical function is *offered*. A player who spends every slot upgrading can still finish a life
  missing that function entirely. The guarantee schedule (Q13) must therefore be expressed against levels,
  not against acquisitions.
- **Augments are unaffected.** P-6 keeps them on level *milestones* and on defined PvE achievements — a
  separate rhythm from the per-level choice, and still a distinct layer.
- The acquisition ledger (M-2…M-4, M-10) must record the **choice type** taken, or "players prefer breadth
  over depth" is unmeasurable.

**Not settled here.** `[O]` **Q15** — offer composition, for both the wild card and the per-level choice: is
the player shown one option of each type, or N drawn from a combined pool? `[O]` **Q16** — upgrade
semantics: a rank advance on the same spell, or a swap to a different stronger spell, and is there a cap?

**Cost to reverse.** Medium. The cadence itself is tuning; the upgrade axis is schema-level, because
acquisition rows must record choice type and upgrade depth from the first migration.

---

## ADR-0010 — Power source: curated client spells for what the player sees, original effects for what warps it

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Answers:** Q3

**Context.** Q3 asked whether power is existing WoW spells granted cross-chassis, original server-authored
effects, or a mix. It sets Phase 1's scope and decides whether this is chiefly a curation effort or a C++
effects effort. The engine was inspected before the options were framed.

**What was verified in this checkout.**
- `[V]` **The world DB overrides *and extends* the spell store server-side, with no client patch.**
  `DBCStores.cpp:355` loads `Spell.dbc` and then the `spell_dbc` SQL table;
  `src/server/shared/DataStores/DBCDatabaseLoader.cpp:79` comments the semantics exactly — *"If exist in DBC
  file override from DB"* — and line 58 grows the index table, so SQL may both rewrite existing spell IDs and
  introduce new ones.
- `[V]` **`SpellInfo` is mutable at load and modules get write access.** `PowerType` and `ManaCost` are plain
  `uint32` (`src/server/game/Spells/SpellInfo.h:386-387`); `ScriptMgr::OnLoadSpellCustomAttr(SpellInfo*)`
  (`src/server/game/Scripting/ScriptMgr.h:534`) passes a non-const pointer to modules; the core already
  mutates SpellInfo this way (`src/server/game/Spells/SpellMgr.cpp:3583`).
- `[V]` **Nothing checks class when spending power.** `Spell::CheckPower` reads the power type from the
  *spell*, not the caster (`src/server/game/Spells/Spell.cpp:7244-7246`). A mismatch fails
  `SPELL_FAILED_NO_POWER` — a clean refusal, not a crash.
- `[V]` **Upgrades are data.** `spell_ranks` is a world DB table (`data/sql/base/db_world/spell_ranks.sql:23`)
  resolved by `GetFirstSpellInChain` / `GetNextSpellInChain` / `GetSpellRank`
  (`src/server/game/Spells/SpellMgr.h:675-679`), and chains are ours to define.
- `[A]` **The client's `Spell.dbc` is untouched and supplies every name, icon and tooltip.** A purely
  server-side spell ID therefore has no presentation — which is why the wiki describes `spell_dbc` as
  "serverside spells which are not to be found in Client DBC files" and why the core uses it for triggered
  effects (`src/server/scripts/EasternKingdoms/ZulGurub/boss_arlokk.cpp:35`). **This is an assumption about
  the 3.3.5a client, not a fact this checkout proves.** Confirming it is Phase 1's job.

**Decision.** Split along the layer boundary ADR-0001 already draws:

| Layer | Made of | Why |
|---|---|---|
| **Abilities**, and the **upgrades** of D-009 | Existing client spell IDs, granted cross-chassis | They live in the spellbook and on the action bar, so they need a real name, icon and tooltip |
| **Talents** and **augments** | Original server-authored effects — module `SpellScript`/`AuraScript`, `spell_dbc` server-side spells, `SpellInfo` mutation at load | They modify existing abilities rather than being cast, so they need no presentation of their own |

**Rationale.** The split is not a compromise between A and B; it falls out of the three-layer model. The layer
that needs presentation takes the constraint that buys presentation for free; the layer that needs freedom
takes the freedom, and pays nothing for tooltips it never needed. D-009 strengthens this further by putting
the entire depth axis onto `spell_ranks`, which is the cheap side.

**Consequences.**

- **D10-a — the presentation budget is fixed and finite.** Every player-facing castable must map to a client
  spell ID. The curated pool is bounded by what the client can already render, not by what we can design.
  This constrains Q13's taxonomy and the pool size, and it should be *measured* in Phase 1, not estimated.
- **D10-b — tooltip truth becomes a curation criterion.** Wherever we alter an existing spell's server-side
  behaviour, its stock tooltip is now wrong. Pillar 4 requires preferring grants whose stock tooltip stays
  true, and treating each divergence as a deliberate, documented exception rather than an accident.
- `[O]` **D10-c — cross-chassis resource mismatch is unresolved, and it gates pool size.** Three routes
  exist: rewrite the spell's cost server-side (the tooltip then lies, straining D10-b); **normalise chassis
  resources** — `Unit::setPowerType` writes the displayed power type
  (`src/server/game/Entities/Unit/Unit.cpp:7040`) and `ChrClasses` is itself SQL-backed via
  `chrclasses_dbc`, so this is feasible server-side; or curate only spells whose resource already matches the
  chassis, which shrinks the pool hard. This is **Q17**.
- **D10-d — Phase 1 must prove both paths and settle the `[A]` above.** Minimum: grant an existing spell
  cross-chassis and observe tooltip, action bar and resource display in a real client; and attach an original
  `AuraScript` effect to an existing ability and confirm it is legible without one.
- **D10-e — none of this requires a core patch.** Module script hooks plus world DB data cover every
  mechanism named above. That satisfies rule 13 far better than expected and should be defended: a core
  patch here needs its own ADR.
- `spell_dbc` and `spell_ranks` are world DB tables, so all such changes travel through `pending_db_*` (T-5)
  and are bound by the SQL guidelines.

**Cost to reverse.** High. It sets Phase 1's scope, decides which skills the project needs, and determines
whether the ability catalogue is a curation artefact or a code artefact.

---

## ADR-0011 — Cross-chassis resource policy: chassis keep their native resource, abilities are curated to match

**State:** Superseded by ADR-0012 · **Date:** 2026-09-04 · **Answers:** Q17 · **Amended:** P-1

> **Superseded the same day by ADR-0012.** The owner supplied a fourth option — every chassis carries
> *every* resource — which the engine turns out to support almost natively. The restriction below is no
> longer in force; the verified findings in it remain accurate and are retained.

**Context.** ADR-0010 left the resource mismatch open as D10-c. Three routes were put forward: normalise every
chassis to one resource and rewrite spell costs; normalise and curate only resource-native spells; or keep
per-chassis resources and curate to match.

**What was verified first.**
- `[V]` **A spell's resource is global to the spell, not the caster.** `SpellInfo::CalcPowerCost`
  (`src/server/game/Spells/SpellInfo.cpp:2803`) is caster-aware for the *amount*, but `PowerType` is read
  from the shared `SpellInfo`, and no script hook exists on power cost. Per-caster resources are impossible
  without a core patch.
- `[V]` Normalisation *would* have been a pure data change — power type comes from `ChrClassesEntry` at
  creation (`Player.cpp:519`) and is re-asserted on form change (`Player.cpp:10778-10779`), and
  `chrclasses_dbc` ships empty as an override-only table (`data/sql/base/db_world/chrclasses_dbc.sql:92-95`).
  This decision declines that option; the finding is retained because Q18 may revisit it.

**Decision (owner).** Each chassis keeps its **native** resource. The curated ability pool for a chassis is
restricted to spells already native to that resource. No `chrclasses_dbc` override, no cost rewriting.

**Consequences.**

- **D11-a — every tooltip stays true.** D10-b is fully satisfied with zero engineering: no rewritten costs, no
  divergence to document, no permanent lie on the cost line. This is the decision's central benefit.
- **D11-b — this amends P-1, deliberately.** P-1 states the chassis "must not lock the player into a role or
  class identity". Under this decision the chassis constrains the entire ability pool for the life, which is
  a real and persistent mechanical identity. `DESIGN_PILLARS.md` §"Using the pillars" requires that a
  requirement blocking something we want be changed by ADR rather than quietly ignored — this is that ADR.
  **P-1 must be rewritten to match**, and Pillar 2's "no class identity" phrasing re-read in this light.
- **D11-c — Q6 is largely pre-answered.** The chassis cannot be "fully hidden": its resource bar is visible
  and it visibly constrains what the player is offered. Q6 now chooses between "acknowledged origin" and
  "light identity", not whether identity exists.
- **D11-d — Q11 becomes far more important.** If chassis fixes the ability pool for a whole life, then a
  vessel permanently locked to one chassis (ADR-0007) locks the player out of most of the game's content
  across every life on that vessel. Q11 is promoted accordingly.
- `[V]` **D11-e — the chassis pools are severely asymmetric, and this is the decision's main risk.**
  **Verified 2026-09-04 from `data/sql/base/db_world/player_class_stats.sql`** (746 rows): `BaseMana` is
  exactly **0 at every level** for warrior, rogue and death knight, and non-zero for the other seven
  (paladin 1512, hunter 1720, priest 1376, shaman 1520, mage 1213, warlock 1373, druid 1244 at level 60).
  So **seven classes use mana and three do not**. Under this decision a mana chassis draws cross-class from
  seven spellbooks and remains richly classless, while a rage, energy or runic chassis draws from exactly
  **one** — which is not a classless build at all, it is that class.
  This also means the entire rage/energy/runic ability space is excluded from *every* chassis unless an
  exemption is granted. See Q18.
- `[O]` **D11-f — are non-mana chassis viable at all under this decision?** If D11-e holds, the options are:
  ship mana-only chassis; accept that rage/energy chassis are narrow "class-like" experiences; or grant those
  chassis a targeted exemption that rewrites costs for them alone. This is **Q18**, and Phase 1 must measure
  the per-chassis pool by role before it can be answered.
- **P-5 role coverage now applies per chassis, not per game.** Every chassis must reach tank, healer, DPS and
  hybrid with multiple archetypes each — a materially harder bar than the original requirement, and one that
  D11-e suggests some chassis may simply fail.
- `FORM_CAT`/`FORM_BEAR`'s hard-coded power switch (`Player.cpp:10763-10773`) is now harmless: it agrees with
  the native resource rather than fighting an override.

**Cost to reverse.** Medium. No schema or code is built on it yet, and the normalisation route stays open
(the mechanism is verified and recorded above). It becomes expensive once a curated catalogue exists.

---

## ADR-0012 — Every chassis carries every resource

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Supersedes:** ADR-0011 · **Answers:** Q18 (dissolves it)

**Context.** ADR-0011 restricted each chassis's ability pool to spells native to its own resource, which
`[V]` D11-e showed produced a severe asymmetry: seven mana classes drawing on seven spellbooks, and warrior,
rogue and death knight drawing on one each. Q18 asked whether non-mana chassis were viable at all. The owner
answered with a fourth option that had not been offered: **every class keeps its own resource *and* gains all
the others — the way a druid already holds mana, rage and energy.**

**The engine already does almost all of this.** Verified in this checkout:

- `[V]` `Unit::GetCreatePowers` (`src/server/game/Entities/Unit/Unit.cpp:12520-12542`) returns **rage 1000,
  energy 100 and runic power 1000 for every unit, with no class check at all**. Only mana is class-dependent,
  via `GetCreateMana()`.
- `[V]` `Player::RegenerateAll` (`src/server/game/Entities/Player/Player.cpp:1809-1843`) unconditionally
  regenerates energy, mana **and** rage for every player. Only runes and runic power are gated to death
  knights.
- `[V]` `Player::Regenerate` (`Player.cpp:1886-1888`) returns early only when `GetMaxPower(power)` is zero.
  There is no class check — a non-zero maximum is the entire gate.
- `[V]` `Spell::CheckPower` (`src/server/game/Spells/Spell.cpp:7244-7246`) reads
  `m_caster->GetPower(spell's PowerType)` and never asks whether that resource belongs to the caster's class.
- `[V]` The one real gap is **mana for warrior, rogue and death knight**, whose `BaseMana` is 0 at every
  level (`data/sql/base/db_world/player_class_stats.sql`, 746 rows). That value flows
  `player_class_stats.BaseMana` → `ObjectMgr.cpp:4791` → `classInfo.basemana` → `SetCreateMana`
  (`Player.cpp:2532`, `2650`) → `GetCreatePowers(POWER_MANA)` → `UpdateMaxPower`.

**Decision.** Every chassis carries every resource. A character has mana, rage and energy simultaneously; the
chassis still determines which resource is *displayed* and its base stat curve, but no longer restricts what
the character may be offered or cast.

**Implementation.** A world DB data change: give classes 1, 4 and 6 a non-zero `BaseMana` curve in
`player_class_stats`, through `pending_db_*` per T-5. `Player::UpdateMaxPower` also exposes
`OnPlayerAfterUpdateMaxPower(Player*, Powers&, float& value)`
(`src/server/game/Scripting/ScriptDefines/PlayerScript.h:467`) with `value` by non-const reference, so the
module can set maxima directly instead if that proves cleaner. **Either route needs no core patch**, keeping
D10-e intact.

**Consequences.**

- **D12-a — the ability pool is whole again.** Every chassis may be offered every curated ability, from every
  class, with **no re-costing and no exemptions**. This keeps ADR-0011's real prize — D11-a, every tooltip
  stays true — while discarding the restriction that bought it. It is strictly better than every option
  offered under Q17 and Q18.
- **D12-b — Q18 is dissolved, not answered.** All ten chassis are viable, and the D11-e asymmetry is gone.
- **D12-c — P-1's ADR-0011 amendment is rolled back in part.** The chassis again does *not* restrict the
  ability pool. It still fixes the displayed resource, base stat curve and armour animation, so it is not
  quite pure scaffolding, but the "real mechanical identity" language of D11-b overstates it now.
- `[A]` **D12-d — the client shows one resource bar, and this is the open problem.** The displayed power
  comes from `UNIT_FIELD_BYTES_0` byte 3 via `Unit::setPowerType` (`Unit.cpp:7040-7042`), and a stock 3.3.5a
  player frame renders a single bar. A character holding three resources can therefore **see only one**, and
  under T-10 there is no AddOn to fix that. Spending a resource the player cannot observe is a direct Pillar 4
  problem. This is **Q19**, and Phase 1 must establish what the client actually does — including whether it
  refuses to send a cast it believes unaffordable.
- **D12-e — three resource economies now run at once and interact.** Rage accrues from combat, energy ticks
  back, mana regenerates on the five-second rule. A build mixing all three has three independent pacing
  curves. That is genuinely novel and is exactly the "legible strangeness" Pillar 4 wants — but it multiplies
  the balance surface, and M-1's resource-economy telemetry must track all three per life from day one.
- `player_class_stats` becomes a **balance-critical table**: BaseMana for classes 1/4/6 is a tuning value that
  changes what those chassis can afford, not a cosmetic fill-in.

**Cost to reverse.** Low-to-medium. It is data, not schema or code, and it deletes work rather than adding
it. The catalogue built on top of it is what would be expensive to revisit.

---

## ADR-0013 — A bundled client AddOn ships the multi-resource frame (amends T-10)

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Answers:** Q19 · **Amends:** T-10

**Context.** ADR-0012 gave every chassis mana, rage and energy at once. D12-d then asked how a player reads
resources the stock frame cannot show, since it renders one bar. The owner's answer: **all resources are
always shown, stacked in the player frame, and the frame is delivered with the game rather than
installed by the player.**

**What was verified.**
- `[V]` The data already reaches the client. `UNIT_FIELD_POWER1..7` and `UNIT_FIELD_MAXPOWER1..7` are all
  `UF_FLAG_PUBLIC` (`src/server/game/Entities/Object/Updates/UpdateFields.h:97-105`,
  `UpdateFieldFlags.cpp:189`). No server work is needed to feed a display.
- `[V]` `UnitPower`, `UnitPowerMax` and `UnitPowerType` are **stock** 3.3.5a client APIs — present in the
  string table of a 3.3.5a client binary. This resolves the assumption previously carried in D12-d.
- `[V]` Arbitrary extra bars in a unit frame are achievable from **pure AddOn Lua**. ShadowedUnitFrames
  registers a module that builds one with `ShadowUF.Units:CreateBar(frame)` driven by stock unit events
  (`Interface/AddOns/ShadowedUnitFrames/modules/druid.lua:1-12`). It is gated to druids by choice, not by
  capability.
- `[V]` A bundled AddOn can live in a plain, loose `Interface/AddOns/` folder — the arrangement used by
  existing 3.3.5a servers that ship addon sets. Bundling therefore requires **no MPQ patch and no modified
  client binary**.

**Decision.** The project ships a **first-party client AddOn** providing a player frame that displays mana,
rage, energy, runic power and runes simultaneously, stacked. It is **bundled and required**, not optional.

**Consequences.**

- **D13-a — this amends T-10**, whose constraint column read "no MPQ patch, no custom DBC, no required
  AddOn". The third clause no longer holds. **The first two still do** and should be defended: this decision
  buys a required AddOn, not a modified client. The AddOn is our own original work and raises no
  redistribution question of its own; packaging is governed separately by rule 12.
- **D13-b — the aura-mirror fallback is dropped.** It existed only to serve a stock client with no AddOn.
  There is now one display path, not two.
- **D13-c — we now own a client-side codebase**, in Lua, with its own release cycle. It must be
  version-locked to the server, since a stale AddOn misreporting resources is worse than no AddOn. This is a
  new maintenance surface, a new skill requirement, and a new QA layer that `QA_STRATEGY.md` does not yet
  cover.
- **D13-d — Pillar 4's stock-client handicap largely lifts, and this cascades.** `DESIGN_PILLARS.md` notes
  Pillar 4 is "harder here than on a server with an AddOn". That is no longer true. In particular **D10-b
  (tooltip truth as a curation criterion) is weakened**: with our own UI we can display our own descriptions,
  which reopens options ADR-0010 and ADR-0011 closed on tooltip-honesty grounds — including re-costing
  spells. Those decisions are **not** reopened here; this records that the ground under them moved.
- **D13-e — the delivery mechanism for the AddOn is settled by ADR-0014**, which is held in the private
  decision store. Nothing in *this* record depends on that choice: the AddOn is our own original work and
  raises no redistribution question of its own.

**Cost to reverse.** Medium. The AddOn is additive and self-contained, but design decisions taken on the
assumption of a rich UI would not survive its removal.

---

## ADR-0014 — Reserved

**State:** Accepted (owner) · **Date:** 2026-09-04 · **Answers:** Q20

This record is held in the project's private decision store rather than in the public fork, per ADR-0005,
which reserved a separate private repository for material that should not be published. The number is
reserved here so the sequence stays intact. **ADR-0016** establishes where that store lives.

---

## ADR-0015 — The server runs in Docker on a dedicated remote host

**State:** Accepted (owner) · **Date:** 2026-09-04

**Context.** The deployment target had not been recorded, so every earlier decision quietly assumed
"somewhere". The owner has stated the intent: the server lives on a **separate machine**, in **Docker**.

**What the repository already provides.** `[V]` Upstream ships first-class Docker support — `docker-compose.yml`
defines `ac-database`, `ac-db-import`, `ac-worldserver`, `ac-authserver`, `ac-client-data-init`, `ac-tools`
and `ac-dev-server`, with named volumes `ac-database`, `ac-client-data`, `ac-build-dev` and `ac-ccache-dev`
(`docker-compose.yml:9-216`). Tooling lives in `apps/docker/` (`docker-cmd.sh`, `Dockerfile`,
`Dockerfile.dev-server`, `entrypoint.sh`). Config templates are in `conf/dist/`, including `env.docker`.

**Decision.** Deploy via upstream's Docker Compose stack on a dedicated host, separate from the development
machine.

**Consequences.**

- **D15-a — no divergence required.** This is upstream's own supported path, so rule 13 is satisfied without
  a core patch or a custom deployment fork. Any future divergence from the shipped compose stack should be
  justified rather than drifted into.
- **⚠ D15-b — the `ac-database` named volume holds every irreplaceable thing this project will ever have,
  and it is one command from destruction.** `docker compose down -v` removes named volumes. T-6 already
  states persistent progression has no reset valve, and D7-a establishes that the prestige reset has no undo.
  Together these mean **a backup and restore policy is a prerequisite to creating the first persistent table,
  not an operational nicety.** Recorded as a mandatory gate in `QA_STRATEGY.md`.
- **D15-c — e2e topology is now an open question.** `AGENTS.md` prefers live-stack e2e "when a local
  auth+world+MySQL stack is available". With the real stack on another machine, we must decide whether
  development runs its own local stack (`ac-dev-server` exists for this), or tests reach the remote one.
  Testing against the host that holds live player state is the option to avoid. This is **Q22**.
- **D15-d — ADR-0003 gains a prerequisite.** azerothMCP reads a populated MySQL. On a remote containerised
  database it needs either network reach to that container or to run on that host. ADR-0003's "install
  immediately before Phase 1" plan now carries a networking and credentials step.
- **D15-e — configuration and secrets need a story.** `conf/*.conf` is gitignored by design, so a remote
  deploy cannot rely on the working tree. `conf/dist/env.docker` is the starting point; how config reaches
  the host, and where credentials live, is undecided and must not default to "copied by hand".
- **D15-f — this host is not the client-distribution host.** ADR-0014's packaging concerns and this game
  server are separate systems with separate exposure. Do not conflate them.
- Builds now have two plausible homes (dev machine vs `ac-dev-server` in-container). `.agents/docs/build.md`
  governs either; the standing rule not to build unless asked is unchanged.

**Cost to reverse.** Low while nothing is deployed. It rises sharply the moment live player state exists on
that host, at which point moving means a migration with real data.

---

## ADR-0016 — The private decision store is a local repository with an encrypted offline backup

**State:** Accepted (owner) · **Date:** 2026-09-05 · **Answers:** Q21

**Context.** ADR-0005 reserved a private repository for material that should not be published but never
established one. ADR-0014 then needed it, and until now that record survived as a single file in a gitignored
directory — no history, no redundancy, one `git clean -xfd` from destruction. Four options were considered: a
private GitHub repository; a self-hosted Forgejo/Gitea instance on the ADR-0015 host; a local repository with
an encrypted offline backup; or encrypting the material inside the public repository.

**Decision.** A **local git repository with no remote**, at `~/Projects/azerothcore-private`, backed up as a
passphrase-encrypted archive copied off-machine manually.

**Rationale.** No third party holds the material at any point. The rejected options each failed on something:
a private GitHub repository would sit on the same platform and account as the public fork; the self-hosted
option depends on a host that is not yet provisioned and whose own backup policy is still open (Q23);
encrypting inside the public repository would publish ciphertext permanently, where a future key compromise
discloses retroactively with no remedy.

**What was set up.** `~/Projects/azerothcore-private` — `git init`, no remote, first commit `d3e9ac1`.
ADR-0014 moved to `decisions/ADR-0014-distribution.md`, byte-for-byte identical (4574 bytes verified with
`cmp`), and the loose copy in the main tree deleted only after the committed copy was confirmed. Commits use
the same `users.noreply.github.com` identity as the public fork, per ADR-0005. `backup.sh` archives the
repository including its history and encrypts it with `gpg --symmetric --cipher-algo AES256`.

**The boundary**, recorded so it is not re-argued per file:

| Private | Public |
|---|---|
| ADR-0014; all packaging and distribution decisions | Game design — vision, pillars, loops, balance |
| The launcher codebase, once it exists | The server module |
| Anything touching client acquisition | The client AddOn — our own original work |

Genuinely ambiguous material goes private. Moving a record private→public later is safe; public→private is
not, because publication is permanent (ADR-0005).

**Consequences.**

- **⚠ D16-a — redundancy is now a manual habit, and that is this decision's weak point.** `backup.sh`
  produces the archive but cannot copy it off the machine or choose a passphrase. Until a backup has actually
  been taken *and a restore tested*, the private record is one disk failure from gone. This is the same
  failure mode D15-b makes a blocking gate for the game database; it deserves the same seriousness here.
- **D16-b — no passphrase is stored anywhere, by design.** `[V]` `gpg` 2.4.9 is installed but the account has
  **no secret key**, so encryption is passphrase-based. Losing the passphrase means losing the backup; it
  belongs in a password manager, not in either repository.
- **D16-c — the two records must cross-reference without the public one disclosing content.** The public log
  keeps a neutral `ADR-0014 — Reserved` placeholder. That convention holds for every future private record.
- **D16-d — migration stays cheap.** If the ADR-0015 host later runs a private git service, adopting it is
  `git remote add` and a push; history carries over. This decision does not foreclose option B, it defers it
  until that host has a proven restore.

**Cost to reverse.** Low. It is a local repository with full history; it can gain a remote, or move, at any
time.

---

## ADR-0017 — Live e2e runs against a full local Docker stack on the development machine

**State:** Accepted (owner) · **Date:** 2026-09-05 · **Answers:** Q22

**Context.** ADR-0015 put the server on a separate host, and `AGENTS.md` prefers live-stack e2e "when a local
auth+world+MySQL stack is available". D15-c asked which stack the tests target.

**What was verified.**
- `[V]` **Live e2e is destructive by design.** `e2e/README.md:220` — *"Live e2e mutates a real realm.
  Cleanup is mandatory."* It creates accounts at **GM level 3** and warns *"Do not reuse real player
  accounts"* (`e2e/README.md:25`); it writes rows to `acore_world` for spawns and game objects and removes
  them with SQL DELETE (`e2e/README.md:228-232`). Correctness depends on cleanup running, so a crashed or
  killed run leaves residue.
- `[V]` **The target is one environment variable from anywhere.** `E2E_AUTH_ADDR`, `E2E_AUTH_DSN`,
  `E2E_CHAR_DSN` and `E2E_WORLD_DSN` default to `127.0.0.1` (`e2e/README.md:48-51`) but accept any host.
- `[V]` `/e2e/.env` is gitignored (`.gitignore:22`) and `e2e/.env.example` ships as the template, so DSNs
  cannot reach the repository by accident.
- `[V]` Upstream provides the local stack: the compose services plus `ac-dev-server`
  (`acore/ac-wotlk-dev-server`, with its own build and `ac-build-dev` volume, `docker-compose.yml:170-208`).

**Decision.** Live e2e runs against a **full local Docker stack on the development machine**, using the
harness's `127.0.0.1` defaults unchanged. **The production host is never a test target.**

**Rationale.** Against T-6 (persistent progression has no reset valve), D7-a (the prestige reset has no undo)
and D15-b (the `ac-database` volume is the only copy), a destructive suite that creates GM accounts must not
be able to reach live player state. Keeping the stack local means production DSNs never exist in the
development environment, so the accident is structurally unavailable rather than merely discouraged — and the
safe configuration is also the default one, which is the cheapest kind of safety to sustain.

**Consequences.**

- **D17-a — the development machine carries the stack:** MySQL, worldserver, authserver and the client-data
  volume. `ac-dev-server` is the supported route. This is a real resource commitment and should be sized
  before Phase 1 rather than discovered during it.
- **D17-b — the operative control is an absence.** Production credentials and DSNs must never be placed in a
  development shell, `e2e/.env`, or any editor run configuration. `.gitignore:22` stops them reaching the
  repository; nothing stops them reaching a shell.
- `[O]` **D17-c — residual risk, accepted knowingly.** A loopback guard in the harness — refusing to run
  unless the target resolves to loopback, overridable by an explicit flag — was offered alongside this option
  and not taken. The control therefore remains **procedural rather than enforced**. Recorded so it is a known
  accepted risk rather than an oversight; adding the guard later is small and cheap, and is the first thing
  to revisit if the stack is ever reconfigured or shared.
- **D17-d — ephemeral-per-run stays available.** `compose up` / test / `compose down -v` gives isolation that
  does not depend on cleanup correctness. Worth adding for CI on top of this decision, not instead of it.
- **D17-e — `QA_STRATEGY.md`'s live-e2e gates become actionable.** Several change types already say live e2e
  is *required*; that was aspirational while no stack existed. It now binds, and `AGENTS.md`'s preference for
  live-stack e2e over unit-sized reasoning applies from Phase 1.
- Scratch work stays in `e2e/local/` (gitignored) per `AGENTS.md`; keepers are promoted to `e2e/suites/` or
  `e2e/smoke/`.

**Cost to reverse.** Low. It is a local environment choice; the harness targets whatever the environment
says.

---

## ADR-0018 — Backups are encrypted logical dumps of the irreplaceable databases, with an automated restore test

**State:** Accepted (owner) · **Date:** 2026-09-05 · **Answers:** Q23

**Context.** D15-b made a *tested restore* a blocking prerequisite to creating the first persistent table:
the database lives in the `ac-database` Docker named volume, `docker compose down -v` destroys it, and T-6
and D7-a leave no reset valve and no undo. D16-a is the same gap in the private decision store.

**What was verified.**
- `[V]` The three databases are not equivalent. Base sizes: `db_world` **297 MB**, `db_characters` **444 KB**,
  `db_auth` **116 KB** (`data/sql/base/`). `acore_world` is **rebuilt from the repository** by the
  `ac-db-import` service (`acore/ac-wotlk-db-import`, `docker-compose.yml:27-54`); `acore_characters` and
  `acore_auth` hold everything that cannot be regenerated.
- `[V]` All tables are InnoDB (`data/sql/base/db_characters/characters.sql`), so
  `mysqldump --single-transaction` yields a consistent snapshot without stopping the server.
- `[V]` AzerothCore ships **no** backup tooling — nothing matching *backup* or *dump* exists under `apps/`.
  This is ours to build.

**Decision.** Scheduled **logical dumps of `acore_auth` and `acore_characters` only**, encrypted, stored
off-host, with the **restore verified automatically inside the same job** — the dump is loaded into a
throwaway MySQL container and asserted against (row counts, and a known character loading). `acore_world` is
excluded because it is reproducible from version control.

**Consequences.**

- **D18-a — excluding `acore_world` makes T-5 load-bearing.** T-5 already requires every schema change to
  travel through `data/sql/updates/pending_db_*`. This decision depends on that: **if `acore_world` ever
  holds anything not reproducible from `data/sql/`, that is a process violation, not a backup gap.** The
  dependency should be *checked* rather than trusted — a periodic comparison of the live world schema against
  the repository is the natural enforcement, and it converts T-5 from a convention into a verified invariant.
- **D18-b — the restore test is the deliverable, not the dump.** A job that produces archives but does not
  load one back does **not** satisfy D15-b. An unverified backup is an assumption with a filename.
- **D18-c — this cannot be implemented or verified yet.** There is no host, no container and no database, so
  a script written today could not have its restore step exercised. Per rule 10 nothing may be claimed as
  working until it has run. **D15-b's gate therefore stands unchanged:** the first persistent table waits on
  this being real and *tested*, not on it being designed.
- **D18-d — passphrase handling matches D16-b.** `[V]` The account has no GPG secret key, so encryption is
  passphrase-based. The passphrase belongs in a password manager, never in either repository, and losing it
  loses the backups.
- `[O]` **D18-e — the acceptable data-loss window is a design question, not only an operational one.**
  ADR-0002's **D2-b** requires staked life endings to be *fully auditable* — cause, location, source,
  timestamp — because a disputed hardcore death is a support incident. A restore that rewinds several hours
  can resurrect a character the realm recorded as dead, or erase the evidence of a legitimate death. The
  backup frequency therefore has to be derived from what a hardcore player would accept losing, not from
  convenience. This is **Q24**.
- **D18-f — the private store can be protected today.** `~/Projects/azerothcore-private/backup.sh` already
  exists; it needs to be run, its output copied off the machine, and verified by decrypting and running
  `git fsck`. **No backup of it has been taken yet**, so that record remains a single copy on one disk.
- Schedule, retention and destination remain operational parameters to fix when the host exists; only the
  *shape* of the policy is settled here.

**Cost to reverse.** Low — it is tooling. The cost of *not having it* before live data exists is unbounded.

---

## Pending decisions

Open questions, in the order they will be asked. Each becomes an ADR when answered.
Full text: `requirements/project-foundation.REQUIREMENTS.md` §14.

**Answered:** Q1 → **ADR-0007**, Q2 → **ADR-0008**, cadence → **ADR-0009**, Q3 → **ADR-0010**
(all 2026-09-04). Between them they raised Q11–Q20. ADR-0011 amended P-1 and was superseded the same day by ADR-0012;
ADR-0013 amends **T-10**, which now permits a required AddOn but still forbids MPQ patches and custom DBC.

| Q | Decision | Blocks | Cost to reverse |
|---|---|---|---|
| Q24 | Acceptable data-loss window, given D2-b requires staked deaths be auditable (D18-e) | Backup frequency; hardcore dispute handling | Medium |
| **Q11** | May prestige change the vessel's chassis? (promoted by D11-d — chassis now fixes the ability pool) | Whether a vessel is locked out of most content for every life | **High** |
| Q15 | Offer composition: one option of each type, or N from a combined pool? (wild card and per-level) | Feel of every choice; draft generator | Medium |
| Q16 | Upgrade semantics: rank advance, or swap to a stronger spell? Is depth capped? (D9-a) | Phase 4; acquisition schema | Medium |
| Q13 | Protected-category taxonomy: which role-functions are guaranteed an offer, and by what level (D8-d) | Phase 4 draft system; P-5 role coverage | Medium |
| Q14 | Decline semantics: may a draft be refused outright, and do unpicked powers return to the pool later in the life? | Feel of every draft; guarantee scheduling | Medium |
| Q4 | Module hosting: separate repo vs vendored in-tree | Phase 1 setup | Medium |
| Q5 | Audience scale and realm openness | Anti-exploit budget, telemetry investment | Medium |
| Q6 | Chassis visibility to the player (see also Q11) | Player-facing framing | Low |
| Q7 | Nerf policy for already-acquired powers | X-11 enforcement credibility | Medium |
| Q8 | Persistent currency shape — and which anchor each unlock hangs from (D7 three-anchor rule) | Phase 2/3 schema | High |
| Q9 | Disconnect / server-fault deaths in staked lives | Player trust; D2-c | Medium |
| Q12 | Vessel deletion vs life records backing account unlocks (D7-c) | Phase 2 schema, data integrity | Medium |
| Q10 | Planning-doc location vs `AGENTS.md` | Process only | Low |

**Next:** Q11, then Q24. Note ADR-0012 defused much of what made Q11 urgent — re-read its
cost-to-reverse before spending a question on it.
