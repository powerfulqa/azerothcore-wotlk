# Project Foundation — REQUIREMENTS (v2)

- **Slug:** `project-foundation`
- **Phase:** RPAC / REFINE
- **Status:** **PROPOSED — awaiting owner approval.** Only §2 decisions are settled.
- **Date:** 2026-09-03
- **Owner:** Chris (sole decision-maker; owns and takes responsibility for the code)
- **Published copy.** Drafting scratch lives in `.claude/plans/project-foundation/` (gitignored);
  this promoted copy in `docs/` is the version of record for review.
- **Supersedes:** `project-foundation.REQUIREMENTS.superseded-v1.md` (v1 recorded a *no-run-reset* direction; the
  project brief reinstated the run loop, so v1's D-001 and its non-goals were wrong and are withdrawn).

**Scope of this phase:** direction, documentation set, architectural stance, QA and balance frameworks,
telemetry and anti-exploit registers, decision process, phase backlog.
**No gameplay code, no schema, no module, no build.**

Legend: `[V]` verified in this repository · `[A]` assumption awaiting confirmation · `[P]` proposal awaiting approval

---

## 1. Repository baseline

### 1.1 Verified state

| Fact | Evidence |
|---|---|
| `[V]` Clean upstream AzerothCore `master` @ `9c700d2a5`, tracking `origin/master`, zero divergence | `git status`, `git rev-parse @{u}` |
| `[V]` No custom modules; `modules/` holds only scaffolding | `ls modules/` |
| `[V]` No `docs/` directory yet | `ls` |
| `[V]` Agent-discipline layer exists and is authoritative | `AGENTS.md`, `.agents/docs/*.md` |
| `[V]` Live-stack e2e harness exists (Go + AzerothGhost v1.0.8), suites for combat, spells, quests, items, instances, guild, social, protocol | `e2e/README.md`, `e2e/suites/` |
| `[V]` SQL writes restricted to `data/sql/updates/pending_db_{auth,characters,world}/` | `AGENTS.md`, `data/sql/updates/` |
| `[V]` `modules/*` is gitignored except scaffolding | `.gitignore:8-13` |
| `[V]` `.claude/plans/**` is gitignored | `.gitignore:68` |

### 1.2 Verified hook and data surface (feasibility evidence)

The single most important technical finding: **the systems in this brief appear implementable from a module,
without core patches.** Register of verified surface, to be cited by future plans rather than re-derived.

**Classless build construction (System A)**

| Need | Verified surface |
|---|---|
| Suppress / repurpose talent points | `PlayerScript::OnPlayerCalculateTalentsPoints(Player const*, uint32& talentPointsForLevel)` — writable out-param |
| Gate talent learning | `OnPlayerCanLearnTalent`, `OnPlayerLearnTalents`, `OnPlayerTalentsReset`, `OnPlayerFreeTalentPointsChanged`, `OnPlayerBeforeInitTalentForLevel` |
| Grant / revoke abilities | `Player::learnSpell(uint32, bool temporary, bool learnFromSkill)`, `Player::addSpell(...)`, `Player::removeSpell(...)`, `Player::SetFreeTalentPoints(uint32)`, `Player::resetTalents(bool)` |
| Observe acquisition | `OnPlayerLearnSpell`, `OnPlayerForgotSpell`, `OnPlayerLevelChanged`, `OnPlayerFirstLogin`, `OnPlayerLogin`, `OnPlayerCreate`, `OnPlayerSave` |
| Starting kit / base stats (data-driven) | world DB `playercreateinfo`, `playercreateinfo_spell_custom`, `playercreateinfo_action`, `playercreateinfo_item`, `playercreateinfo_skills`, `player_class_stats` |

**Roguelite run and life-end (System B)**

| Need | Verified surface |
|---|---|
| Detect death | `OnPlayerJustDied`, `OnPlayerKilledByCreature`, `OnPlayerReleasedGhost` |
| **Enforce** hardcore (block revival) | `bool OnPlayerCanResurrect`, `bool OnPlayerCanRepopAtGraveyard` — boolean gates, so a module can actually prevent resurrection |
| Persist across lives | `OnPlayerDelete(ObjectGuid guid, uint32 accountId)`, `OnPlayerDeleteFromDB(CharacterDatabaseTransaction, uint32 guid)`, `AccountScript::OnAccountLogin`, `OnBeforeAccountDelete` |
| Level-milestone triggers | `OnPlayerLevelChanged`, `OnPlayerSetMaxLevel`, `OnPlayerCanGiveLevel` |
| Achievement-based triggers | `AchievementScript`, `AchievementCriteriaScript`, `OnPlayerAchievementSave`, `OnPlayerCriteriaSave` |

**Balance, pacing and telemetry**

| Need | Verified surface |
|---|---|
| Combat output modification | `UnitScript::OnDamage`, `OnHeal`, `OnAuraApply`, `OnAuraRemove`, `OnUnitDeath`, `OnUnitEnterCombat`, `OnUnitExitCombat`, `OnBeforeRollMeleeOutcomeAgainst` |
| Stat / scaling modification | `OnPlayerCustomScalingStatValueBefore`, `OnPlayerCustomScalingStatValue`, `OnPlayerAfterUpdateMaxPower` |
| XP pacing (multiplier) | `FormulaScript::OnBaseGainCalculation`, `OnGainCalculation`, `OnGroupRateCalculation`; `OnPlayerGiveXP`, `OnPlayerQuestComputeXP`, `OnPlayerBeforeGetLevelForXPGain` |
| **XP pacing (curve shape)** | world DB table `player_xp_for_level` (`ObjectMgr.cpp:4928`) — the XP requirement per level is fully data-driven |
| Level cap | config `MaxPlayerLevel` (`worldserver.conf.dist:2119-2125`, default 80, range 1–255); `DEFAULT_MAX_LEVEL 80`, `STRONG_MAX_LEVEL 255` (`src/server/shared/DataStores/DBCEnums.h:35,43`) |
| Loot / economy observation | `OnPlayerLootItem`, `OnPlayerAfterCreatureLoot`, `OnPlayerAfterCreatureLootMoney`, `OnPlayerMoneyChanged`, `OnPlayerCreateItem`, `OnPlayerQuestRewardItem` |

`PlayerScript` exposes **187** registered hooks in total.

**Consequence for the 6-hour target:** because `player_xp_for_level` is data-driven, run pacing can be
achieved by *reshaping the curve* rather than only inflating `Rate.XP.*`. This satisfies the brief's
"do not rely only on a huge experience-rate multiplier" constraint by construction.

### 1.3 Verified hard constraints (client-side / immovable without a client patch)

| Constraint | Evidence | Consequence |
|---|---|---|
| `[V]` Talent tabs are class-masked | `TalentTabEntry.ClassMask`, `DBCStructure.h:1946` | The stock talent UI cannot present a classless tree. Classless talents need a different delivery channel. |
| `[V]` Primary resource is class-bound | `ChrClassesEntry.powerType`, `DBCStructure.h:656` | A character has exactly one of mana/rage/energy/runic, fixed by its technical class. This is why a **chassis** exists; it is a client constraint, not a design preference. |
| `[V]` Character creation data is server-side | `playercreateinfo*`, `player_class_stats` | Starting kit and stat curves are ours to redefine in SQL. |

### 1.4 Items requiring verification before they may be relied upon `[A]`

Explicitly flagged so no plan treats them as established:

- **V-1** Which level-60 raid and dungeon encounters exist in the 3.3.5a client, and at what tuning. The
  classic raids were re-tuned across expansions; their 3.3.5a state must be measured, not remembered.
  Blocks the endgame and solo-raid assessment.
- **V-2** Cross-chassis spell behaviour. Server side partly verified by ADR-0010/0011; the client side and
  the class→resource mapping remain unverified (no `data/dbc/` in this checkout). Scaling, stance/form
  requirements, resource assumptions, pet and
  aura assumptions, and which spells silently no-op. Blocks the entire power catalogue (see R-2, Phase 1).
- **V-3** Whether classic 1–60 quest and dungeon content in the 3.3.5a world DB supports a coherent
  solo levelling path at the intended pace without gaps.
- **V-4** Behaviour of achievements, gear and reputations under a 60 cap on a WotLK client.

---

## 2. Confirmed decisions

**D-001r — Direction: persistent-progression roguelite with per-life classless builds.**
*(Owner, 2026-09-03. Supersedes v1's D-001, which wrongly excluded the run layer.)*

A **life** is a level 1→60 run. Within a life the player constructs a classless build from curated
cross-class abilities, talent-like passives and roguelite augments. Run-scoped progress resets when a life
ends; account-scoped progression persists and compounds across lives, favouring **agency before raw power**.

**D-002 — Life-end model: voluntary prestige, plus opt-in stakes.** *(Owner, 2026-09-03)*

- **Level 60 is a home, not a finish line.** The default life ends only when the player *chooses* to
  prestige. This is what makes the 3-player endgame and repeated high-mastery solo-raid attempts possible;
  a death-ends-life default would contradict both.
- **Optional stake modifiers are declared at the start of a life** (e.g. hardcore, challenge conditions)
  and grant additional persistent rewards proportionate to the risk taken.
- Therefore the life record carries a **ruleset/modifier field** from day one, and there are **two death
  paths** (default and staked) to design, implement, test and balance.

**Derived requirements from D-002:**
- **D2-a** Stake rewards must make hardcore *attractive but never mandatory* — no persistent unlock may be
  reachable only through a staked life, or the "optional" framing is false.
- **D2-b** A staked life's ending must be auditable. Death cause, location, source and timestamp are
  recorded, because a disputed hardcore death on a private server is a support incident.
- **D2-c** Disconnect and server-fault deaths need a stated policy. Silence here is a player-trust failure.

---

## 3. Goals

- **G-1** An approved, written direction any future phase can be checked against.
- **G-2** The documentation and ADR structure in §10, proposals clearly marked, no invented decisions.
- **G-3** Technical stance: module-first, core patches last-resort and documented; verified achievable per §1.2.
- **G-4** QA bar: what "verified" means per change type, and which layer owns what.
- **G-5** Balance framework: dimensions, target bands, and the telemetry required to tune with evidence.
- **G-6** Anti-exploit register covering every item the brief lists as unacceptable.
- **G-7** Repository operating model: branching, upstream rebase cadence, module hosting, SQL discipline.
- **G-8** An ordered phase backlog so no phase begins without a stated reason it is next.

---

## 4. Non-goals (this phase)

- **NG-1** No gameplay code, C++, or module scaffolding.
- **NG-2** No schema and no migration files, not even in `pending_db_*/`.
- **NG-3** No build, configure or server run.
- **NG-4** No final balance numbers, no ability/talent/augment catalogue.
- **NG-5** No content authoring.
- **NG-6** No modification of the game client binary or its data: no MPQ patch, no custom DBC.
  *(Amended by ADR-0013: a required first-party AddOn is permitted and is not a client modification.)*
- **NG-7** **No PvP scope**, at any point in the project. PvP balance is explicitly not a design dimension.
- **NG-8** No third-party server's code, database, UI, assets, branding, terminology, exact mechanics,
  balance values, custom content, text or proprietary designs. Inspiration is limited to publicly-observable
  genre principles; every artefact is original.
- **NG-9** No commercial activity, monetisation, paid access, or donation-for-power.

---

## 5. Player-facing behaviour — proposed shape `[P]`

### 5.1 The life

- **P-1 Chassis.** A character has an underlying technical class fixing resource type, armour animation and
  base stat curve (§1.3). **Settled by ADR-0012 (2026-09-04), which superseded ADR-0011:** every chassis
  carries *every* resource, so the chassis does **not** restrict the ability pool — any chassis may be
  offered any curated ability. It still fixes the **displayed** resource, the base stat curve and armour
  animation, so it is not quite pure scaffolding, but it must not lock the player into a role or class
  identity and must not be the primary thing a build is "about". Visibility to the player is **Q6**; the
  displayed resource bar means it cannot be entirely hidden.
- **P-2 Starting kit.** Every life begins with a small, *viable* combat kit — able to kill, survive and
  progress solo from level 1. A dead or unusable starting kit is a defect, not a bad roll (see X-1).
- **P-3 Acquisition through levelling.** Curated cross-class **abilities** (active) and **talents**
  (passive) are acquired as the player levels. The acquisition model is settled by **ADR-0008**: a
  choice-of-N draft (N starts at 3, tunable) in which protected categories guarantee a role-critical
  function is *offered* — never forced — and an account-earned reroll budget is spent within the life.
  **ADR-0009** sets its cadence: a wild card selection at life start, then at every level a choice between a
  new ability, an **upgrade** to one already held, or a talent. The upgrade axis is depth, not a fourth
  power layer, and it competes for the same slot as breadth.
- **P-4 Eligibility and tags.** Every ability, talent and augment carries tags and eligibility rules so
  incompatible, useless or exploit-prone combinations cannot be offered. This is a *system requirement*,
  not per-item curation, because per-item curation does not scale.
- **P-5 Role coverage.** Tank, healer, DPS and hybrid builds must all be reachable, and **multiple distinct
  archetypes must be viable for each role** (not one per role).
- **P-6 Augments.** Roguelite augments are earned at level milestones and, potentially, from defined PvE
  achievements. They *warp* the build — resource changes, cooldown changes, area effects, triggers,
  scaling transformations, defensive conversions, risk/reward trades. They are a distinct layer from
  abilities and talents and must never be presented as the same thing.
- **P-7 Legibility.** A player must be able to explain why their build works. Confusing power is churn.

### 5.2 Pacing

- **P-8** First life: ≈6 hours 1→60 for an informed ordinary player, solo, with no persistent unlocks.
- **P-9** Early lives feel like a recognisable Vanilla-style world journey: questing, exploration, elites,
  gear and dungeon runs all matter.
- **P-10** Later lives are progressively faster through persistent unlocks, build agency and stronger
  synergies — *not* primarily through an XP-rate multiplier (§1.2).

### 5.3 Life end and persistence

- **P-11** Prestige is a deliberate, confirmed action at 60, with the consequences shown before commitment.
- **P-12** Optional stake modifiers are declared at life start, visible for the life's duration, and
  immutable once chosen.
- **P-13** Persistent rewards favour **agency**: better starting options, wider drafts, curated pools,
  rerolls, build slots, controlled progression modifiers. Any *raw power* reward requires caps, pacing,
  diminishing returns and telemetry before it ships.
- **P-14** A returning player must be able to see what they have permanently unlocked and what it changes.

### 5.4 Endgame

- **P-15** Default balance target: a 3-player group of one tank, one healer, one DPS at level 60.
- **P-16** Group play stays rewarding even though high-progression solo play exists.
- **P-17** At late progression, specialised builds may solo *selected* raid encounters. **Every boss is
  individually assessed for solo feasibility** and the assessment is recorded.
- **P-18** Multi-player-only mechanics get deliberate, documented adaptations. An encounter that becomes
  soloable through an unintended bypass is a **defect**, not a feature.

---

## 6. Technical requirements

- **T-1 Module-first.** All custom behaviour lives in a module. Core files change only where no hook
  exists; each such change requires an ADR and a rebase-risk note.
- **T-2 No invented surface.** Every hook, table, column, config and command must be verified in this
  checkout before appearing in a plan. §1.2 is the register; §1.4 is what is *not* yet verified.
- **T-3 Upstream compatibility.** Track `origin/master`; rebase on a defined cadence; keep and report the
  core diff against upstream.
- **T-4 Data-driven.** Tunables live in module config and custom DB tables, never in C++ literals, so
  balance changes need no rebuild. This is a hard requirement given §8's evidence-based balancing.
- **T-5 Schema discipline.** Custom tables carry a project prefix, never collide with AzerothCore names,
  land in `pending_db_*/`, and each migration states its rollback.
- **T-6 Persistence lifecycle.** Every custom table defines behaviour for: creation, save point, life end,
  prestige, character deletion (`OnPlayerDelete` supplies `accountId`), account deletion, and GM repair.
- **T-7 Combat-path performance budget.** `UnitScript::OnDamage`/`OnHeal` run at very high frequency;
  augments that hook them need a bounded, *measured* per-call cost.
- **T-8 Observability from day one.** Structured logging, GM inspection commands, and an audit trail for
  every ability/talent/augment grant, every persistent-currency movement, and every life end.
- **T-9 Idempotent grants.** Every acquisition path must be idempotent and transactional, or duplicate
  rewards become an exploit class (X-8).
- **T-10 Client expressibility.** **Amended by ADR-0013 (2026-09-04).** Every player-facing affordance must
  trace to a stock-client mechanism — gossip menu, spell, aura, item, action bar, chat, tooltip — **or to the
  project's own bundled AddOn** — at PLAN time, not ACT time. A **required AddOn is now permitted**; MPQ
  patches and custom DBC remain forbidden. See rule 12 for the packaging and distribution stance.

---

## 7. Telemetry requirements

Balancing is evidence-based (§8), so telemetry is a **prerequisite**, not an afterthought. The brief's
list, restated as a requirement set. Each metric names the layer that produces it.

| # | Metric | Source layer |
|---|---|---|
| M-1 | First-run and repeat-run 1→60 time (wall-clock and played) | life record |
| M-2 | Ability pick rates | acquisition ledger |
| M-3 | Talent pick rates | acquisition ledger |
| M-4 | Augment pick rates | acquisition ledger |
| M-5 | Build distribution and extreme outliers | build snapshot at milestones |
| M-6 | Damage, healing, mitigation, threat, resource performance | combat aggregation |
| M-7 | Deaths, wipes, and reset causes | death/life-end audit (D2-b) |
| M-8 | Dungeon, raid and solo-raid completion rates | encounter records |
| M-9 | Currency generation and spending | economy ledger |
| M-10 | Reward acquisition | acquisition ledger |
| M-11 | Exploit indicators (§9) | anti-exploit counters |

- **TM-1** Metrics must be queryable offline (SQL) without attaching a debugger to a live server.
- **TM-2** Build snapshots must be reconstructable after the fact, or M-5 and M-2/3/4 are unanalysable.
- **TM-3** Telemetry volume must have a stated retention and growth budget; a per-damage-event table on a
  live realm is not viable and must not be proposed casually.

---

## 8. Balance framework

### 8.1 Dimensions tracked

Damage · survivability · healing · threat/mitigation · utility · mobility · crowd control · resource
economy · level and gear scaling · acquisition effort · skill ceiling · group value · exploit potential.
**PvP is excluded** (NG-7).

### 8.2 Target performance bands

To be specified in `docs/BALANCE_FRAMEWORK.md` by: content tier · role (tank/healer/DPS/hybrid) ·
solo vs 3-player · encounter duration · gear level · **persistent-progression stage**. The last axis is
unique to this project and is the one most likely to be forgotten.

### 8.3 Hypotheses (stated to be falsified with data, not defended by argument)

- **H-1** Eligibility rules and tags (P-4) are sufficient on their own to prevent dead builds, without
  needing a manual per-combination blocklist.
- **H-2** Acquisition effort is a genuine balance lever: harder-to-obtain powers may be stronger *provided*
  acquisition is not repeatably farmable.
- **H-3** Persistent *agency* rewards (P-13) will produce most of the later-life speed gain; raw-power
  rewards will be needed only sparingly. If false, the whole persistent economy needs redesign.
- **H-4** The dominant exploit surface is acquisition-rate abuse and reset abuse, not raw power.
- **H-5** Build legibility (P-7) matters more for retention than build depth.
- **H-6** Stock 3.3.5a level-60 content will be trivialised by an optimised cross-chassis build; some form
  of scaling or retuning will be required. Magnitude unknown until V-1/V-2.
- **H-7** A 3-player tank/healer/DPS target is achievable without per-encounter bespoke tuning. This is
  optimistic and should be tested early.

---

## 9. Anti-exploit register

Every item the brief lists as unacceptable, with the layer that prevents it. Expanded in
`docs/BALANCE_FRAMEWORK.md` and enforced by e2e (§12).

| # | Must not occur | Primary prevention |
|---|---|---|
| X-1 | Dead starting kits | P-2 as an acceptance criterion; automated viability check over every legal starting kit |
| X-2 | Unusable ability rolls | P-4 eligibility rules + the D-008 draft: the player can decline any offered power |
| X-3 | Infinite resource loops | Augment interaction rules; resource-generation caps; combat-log assertions |
| X-4 | Recursive damage/healing loops | Trigger-depth limit and re-entrancy guard in the augment trigger system |
| X-5 | Permanent invulnerability | Mitigation stacking caps; duration floors; explicit uptime assertions |
| X-6 | Infinite crowd control | CC uptime caps; diminishing returns preserved for PvE |
| X-7 | Pet duplication | Guardian/summon hooks (`OnPlayerBeforeGuardianInitStatsForLevel`) + summon count caps |
| X-8 | Reward duplication | T-9 idempotent, transactional grants + acquisition audit trail |
| X-9 | Reset abuse | Life-end and prestige rules; cooldowns/eligibility on prestige; M-7 monitoring |
| X-10 | Trading / boosting / economy exploits | Persistent progression is account-scoped and non-tradeable; group-acquisition rules; M-9 monitoring |
| X-11 | A single mandatory best build | P-5, M-2…M-5 outlier detection, and a stated nerf policy for already-acquired powers |

**Policy gap to close:** X-11 requires a decided answer to "what happens to a player who already owns a
power we nerf?" — surfaced as **Q7**.

---

## 10. Proposed documentation set `[P]`

Created **after approval**, in `docs/` (version-controlled, unlike `.claude/plans/`):

| File | Purpose |
|---|---|
| `docs/PROJECT_VISION.md` | Fantasy, audience, why play this, success metrics, legal/non-commercial stance |
| `docs/DESIGN_PILLARS.md` | 3–5 pillars used to accept or reject every future feature |
| `docs/CORE_GAME_LOOP.md` | Core / session / life / persistent / endgame loops under D-001r and D-002 |
| `docs/BALANCE_FRAMEWORK.md` | Dimensions, target bands, tuning method, telemetry (§7), anti-exploit register (§9) |
| `docs/TECHNICAL_ARCHITECTURE.md` | Module boundary, hosting, namespacing, verified-surface register (§1.2), rebase policy, performance budgets |
| `docs/QA_STRATEGY.md` | Test layers, mandatory gates per change type, evidence rules |
| `docs/CONTENT_PIPELINE.md` | How an ability / talent / augment / encounter goes from idea to live, with review gates |
| `docs/DECISION_LOG.md` | ADR index; D-001r and D-002 are its first entries |

---

## 11. Proposed phase backlog `[P]`

Ordered so that each phase is independently valuable, stoppable, and placed before anything that depends on it.

| # | Phase | Why it is in this position |
|---|---|---|
| 0 | **Foundation docs** (this) | Everything else needs an approved direction |
| 1 | **Cross-chassis spell spike** (V-2, R-2) | Narrow, empirical, on a live stack. It can *invalidate the entire power model*, so it must come before any catalogue design. Highest information-per-hour in the project. |
| 2 | **Life lifecycle skeleton** | Life record, prestige, stake-modifier field, account-scoped persistence anchor. No powers. Establishes the schema everything else attaches to, while the schema is still cheap to change. |
| 3 | **Telemetry substrate** (§7) | Must exist before any balance claim. Needs Phase 2 to have something to measure. |
| 4 | **Classless acquisition MVP** (System A) | Abilities + talents under the D-008 draft, one role proven end-to-end |
| 5 | **Augment layer** (System B) | Depends on 4; needs the trigger-depth guards from X-3/X-4 designed in, not retrofitted |
| 6 | **Levelling curve retune** | Needs 3 to measure against the 6-hour target |
| 7 | **Endgame assessment** (V-1, P-15–P-18) | Needs a real build to assess encounters against |

---

### 11.1 Tooling dependency: azerothMCP `[P]`

`https://github.com/azerothcore/azerothMCP` — official AzerothCore org, GPL-2.0, Python, active
(last push 2026-04-15). Verified via `gh api`, not assumed.

It is a **read-only** MCP server over the world/characters/auth databases plus SmartAI, spell, quest, item,
condition and waypoint tooling (`query_database`, `get_table_schema`, `search_spells`,
`lookup_spell_names`, `diagnose_quest`, `search_creatures`, `get_smart_scripts`, `trace_script_chain`, …),
with progressive disclosure to keep token cost down.

**Why it matters here:** three of our four unverified items in §1.4 are database-shaped questions.
- **V-2** (cross-chassis spell behaviour) needs bulk spell inspection — Phase 1's core activity.
- **V-1** (which level-60 encounters exist at what tuning) needs creature/template queries.
- **V-3** (1–60 quest path coverage) is exactly what `diagnose_quest` and quest search are for.

**When it makes sense:** immediately **before Phase 1**, and not earlier. Two blocking reasons:
1. It is a read-only client over a **populated MySQL** (`acore_world`, `acore_characters`, `acore_auth`).
   This project has no build and no database yet, so today it would have nothing to read.
2. Phase 0 is documentation-only (AC-9 forbids touching anything outside `docs/` and `.claude/plans/`).

**Constraints to respect when we do install it:**
- Read-only by design. It is an *inspection* tool; it must never become a write path for migrations.
  All schema change continues to go through `pending_db_*/` (T-5).
- It does not relax T-2. Facts it reports still get cited with the query that produced them.
- Enable `ENABLE_SOURCE_CODE` / `ENABLE_WIKI` only if needed; both are off by default and both cost tokens.
- Recorded as an ADR in `docs/DECISION_LOG.md` since it becomes part of the verification workflow.

**Action:** deferred to a Phase-1 prerequisite step, alongside standing up the local stack that
`e2e/` already needs. No installation in this phase.

## 12. Test approach

For **this** phase, verification is documentary: AC-1…AC-9 (§13) are checkable by inspection.

Project-wide layering, to be formalised in `docs/QA_STRATEGY.md`:

| Layer | Location | Owns |
|---|---|---|
| Unit | `src/test/` (GoogleTest) | Eligibility rules, tag matching, budget arithmetic, curve maths, config validation |
| Live e2e | `e2e/suites/` (Go, AzerothGhost) | Grant/removal, persistence across logout, prestige, death paths, combat effects, multi-character interaction |
| Scratch e2e | `e2e/local/` (gitignored) | Debugging only; never committed; keepers promoted |
| SQL validation | `pending_db_*/` on a scratch DB | Applies cleanly, idempotent, rollback proven |
| Manual in-game | Owner | Feel, legibility, UX under a stock client |
| Exploit/abuse | Live e2e + review | Every row of §9 needs at least one negative test |
| Performance | Instrumented live run | T-7 combat-path budget; TM-3 telemetry growth |
| Balance regression | Telemetry queries | M-1…M-11 against §8.2 bands |

**Evidence rule.** No build, test, migration or in-game check is reported as passing unless it was actually
run and its output shown. This binds me without exception.

---

## 13. Acceptance criteria for the foundation phase

- **AC-1** `docs/` contains the eight documents in §10, each marked *Proposed* or *Approved*.
- **AC-2** `docs/DECISION_LOG.md` contains D-001r and D-002, plus one ADR per approved answer to §14.
- **AC-3** A verified-surface register exists (from §1.2) plus an explicit *unverified* list (§1.4).
- **AC-4** `docs/QA_STRATEGY.md` states mandatory verification per change type and the evidence rule.
- **AC-5** `docs/BALANCE_FRAMEWORK.md` contains §8's dimensions, the band axes including progression stage,
  the §7 telemetry table and the §9 anti-exploit register.
- **AC-6** `docs/TECHNICAL_ARCHITECTURE.md` records the module/core boundary, module hosting decision,
  namespacing, rebase cadence and performance budgets.
- **AC-7** The §11 phase backlog is approved or amended.
- **AC-8** `docs/PROJECT_VISION.md` survives an explicit "why play this rather than retail or another
  classless/roguelite server?" test, and restates NG-8/NG-9.
- **AC-9** No file outside `docs/` and `.claude/plans/` is modified. No SQL. No C++. No build.

---

## 14. Open questions — sequential, one at a time

Ordered by how much each constrains everything downstream.

- **Q1 — ANSWERED 2026-09-04 → ADR-0007.** A life is neither purely a character nor a bare reset: the
  character is a persistent **vessel**, prestige resets it in place to level 1, and each life is a
  first-class row carrying stake modifiers, timestamps, outcome and audit trail. Raised **Q11** and **Q12**.
- **Q2 — ANSWERED 2026-09-04 → ADR-0008.** Choice-of-N draft with protected categories that guarantee a
  role-critical function is *offered* rather than possessed, plus an account-earned, life-spent reroll
  budget. N starts at 3 and is a tuning value; offers are stored as rows, never fixed columns. Raised
  **Q13** and **Q14**.
- **Q3 — ANSWERED 2026-09-04 → ADR-0010.** Split by layer: abilities and upgrades are existing client
  spell IDs granted cross-chassis (they need spellbook presentation); talents and augments are original
  server-authored effects (they modify abilities and need no tooltip). Verified that this needs **no core
  patch** — module hooks plus world DB data suffice. Raised **Q17**.
- **Q4** — **Module hosting**: separate repository under `modules/` (clean upstream history, standard AC
  practice) or vendored in-tree via `!modules/mod-<name>` (single repo, simpler solo)?
- **Q5** — **Levelling path**: is the full classic 1–60 world open, or curated/gated to guarantee pacing and
  coherence (V-3)?
- **Q6 — ANSWERED 2026-09-05 → ADR-0019.** "A light identity that becomes irrelevant." It cannot be hidden,
  because early stat differences are visible and consequential, but it must never be presented as a class
  and by level 60 it is inert. Player-facing language describes an origin, not a profession.
- **Q7** — **Nerf policy** (X-11 gap): what happens to a player who already owns a power we nerf — retroactive,
  grandfathered, or refunded?
- **Q8** — **Persistent currency shape**: one currency, several, or direct unlock-on-achievement with no
  currency at all?
- **Q9** — **Disconnect/server-fault deaths in staked lives** (D2-c): forgiven, appealed, or final?
- **Q10** — **Planning-doc location**: keep `.claude/plans/` per your instruction and amend `AGENTS.md`, or
  switch to `.agents/plans/` per the repo's existing convention? Both are gitignored.
- **Q11 — ANSWERED 2026-09-05 → ADR-0019**, together with Q6. **Prestige may change the chassis freely.**
  The premise had dissolved: ADR-0012 and ADR-0013 left the chassis restricting neither the ability pool nor
  the displayed resource, and `[V]` gear proficiency turns out to be skill-based rather than class-locked
  (`PlayerStorage.cpp:2330-2341`), so nothing locks a vessel out of content. The chassis is now a **starting
  nudge** whose stat curves converge by level 60.
- **Q12** — **Vessel deletion vs life records** (D7-c). If life records back account-level unlocks, deleting a
  vessel must not orphan or destroy them. Are life records reparented to the account, retained orphaned, or is
  vessel deletion restricted?
- **Q13** — **Protected-category taxonomy** (D8-d). Which role-functions are protected — healing, defensive,
  AoE, mobility, sustain? — and by what level must each have been offered? Too few and dead builds return;
  too many and every life converges on the same shape.
- **Q14** — **Decline semantics.** May a player refuse a draft outright and bank nothing, and do unpicked
  powers return to the pool later in the same life? Determines whether guarantees are one-shot, and changes
  how every draft feels.
- **Q15** — **Offer composition.** For both the wild card and the per-level choice: is the player shown one
  option of each type (ability / upgrade / talent), or N options drawn from a combined pool? The first makes
  every level a comparison across kinds; the second makes the kinds compete for slots.
- **Q16** — **Upgrade semantics** (D9-a). Is an upgrade a rank advance on the same spell, or a swap to a
  different, stronger spell? Is upgrade depth capped, and does a capped ability stop being offered?
- **Q17 — ANSWERED 2026-09-04 → ADR-0011.** Chassis keep their native resource; abilities are curated to
  match. Every tooltip stays true at zero engineering cost, but the chassis now carries real mechanical
  identity — this **amends P-1** above, by owner decision and per the pillars' own escape clause. Raised
  **Q18**.
- **Q18 — DISSOLVED 2026-09-04 by ADR-0012.** The question presupposed ADR-0011's restriction. Since every
  chassis now carries every resource, all ten are viable and the D11-e asymmetry is gone.
- **Q19 — ANSWERED 2026-09-04 → ADR-0013.** All resources are shown at once, stacked in the player frame,
  delivered by a **bundled, required first-party AddOn**. `[V]` Verified achievable in pure AddOn Lua with
  stock APIs — no MPQ patch and no modified client. This **amends T-10** and raised **Q20**.
- **Q20 — ANSWERED 2026-09-04 → ADR-0014**, which is held in the project's private decision store per
  ADR-0005 rather than in this public record.
- **Q21 — ANSWERED 2026-09-05 → ADR-0016.** A local repository with no remote, backed up as a
  passphrase-encrypted archive taken off-machine by hand, so no third party holds the material. The
  public/private boundary is recorded in that ADR. Its known weakness is D16-a: redundancy is a manual habit
  until a restore has actually been tested.
- **Q22 — ANSWERED 2026-09-05 → ADR-0017.** Live e2e runs against a **full local Docker stack on the
  development machine**, using the harness's loopback defaults; the production host is never a test target.
  `[V]` Justified because the suite is destructive by design — it creates GM level 3 accounts and writes
  world-DB rows with mandatory cleanup (`e2e/README.md:25`, `:220`, `:228-232`). Residual risk **D17-c**: a
  loopback guard was offered and declined, so the control is procedural rather than enforced.
- **Q23 — ANSWERED 2026-09-05 → ADR-0018.** Encrypted logical dumps of `acore_auth` and `acore_characters`
  only, scheduled and stored off-host, with the restore asserted automatically in the same job.
  `acore_world` is excluded because `ac-db-import` rebuilds it from `data/sql/` — which makes T-5
  load-bearing (D18-a). Per D18-c this is designed but not yet exercisable, so D15-b's gate stands.
- **Q24** — **Acceptable data-loss window** (D18-e). D2-b requires staked life endings to be fully
  auditable, so a restore that rewinds hours could resurrect a character the realm recorded as dead, or
  destroy the evidence of a legitimate one. Backup frequency must follow from what a hardcore player would
  accept losing, not from convenience.
- **Q25 (next)** — **Is proficiency granted, or acquired?** (D19-d) `[V]` Armour and weapon proficiency is
  skill-based and grantable — via `playercreateinfo_skills` (world DB) or `SPELL_EFFECT_PROFICIENCY`
  (`SpellEffects.cpp:2319`) — so it is now a choice rather than an inherited default. Either every chassis
  receives every proficiency at life start, making armour type purely cosmetic; or proficiency is itself an
  **acquisition** the player drafts, which would make gear a real axis of the build and add a draft dimension
  that is neither ability, talent, upgrade nor augment.

---

## 15. Requested approval

1. **§2** — D-001r and D-002 captured correctly?
2. **§5** — is P-1…P-18 the game you want, or which points are wrong?
3. **§4** — non-goals correct, particularly NG-7 (no PvP, permanently)?
4. **§9** — anti-exploit register complete, or is something missing?
5. **§10** — approve the documentation set?
6. **§11** — approve the phase order, especially the spike-first placement of Phase 1?
7. **Q1–Q22** — answered since this document was written; see `../DECISION_LOG.md` for ADR-0007
   onward. **Q23** is next.
