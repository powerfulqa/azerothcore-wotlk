# Technical Architecture

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-03
**Purpose:** The authoritative register of what this project may rely on, what it may not, and the rules
that keep it maintainable against upstream AzerothCore.

**Legend:** `[V]` verified in this checkout · `[A]` assumption · `[P]` proposal · `[O]` open question

---

## 1. Baseline

`[V]` Clean upstream AzerothCore `master` @ `9c700d2a5`, tracking `origin/master`, zero divergence at
foundation time. Target client: WoW 3.3.5a, **unmodified**.

---

## 2. Architectural rules

| # | Rule | Rationale |
|---|---|---|
| T-1 | **Module-first.** All custom behaviour lives in a module. Core files change only where no hook exists, and each such change requires an ADR plus a rebase-risk note | Core divergence is the main long-term maintenance cost of a private server |
| T-2 | **No invented surface.** Every hook, table, column, config and command must be verified in this checkout before appearing in a plan | §3 is the register; §4 is what is explicitly *not* verified |
| T-3 | **Upstream compatibility.** Track `origin/master`; rebase on a defined cadence; keep and report the core diff | A growing diff is a measurable warning sign |
| T-4 | **Data-driven.** Tunables live in module config and custom DB tables, never in C++ literals | Balance changes must not require a rebuild (Pillar 5) |
| T-5 | **Schema discipline.** Custom tables carry a project prefix, never collide with AzerothCore names, land in `data/sql/updates/pending_db_*/`, and each migration states its rollback | `[V]` `base/`, `archive/` and merged `db_*/` are immutable per `AGENTS.md` |
| T-6 | **Persistence lifecycle.** Every custom table defines: creation, save point, life end, prestige, character deletion, account deletion, GM repair | Persistent state has no reset valve; a schema mistake follows every account forever |
| T-7 | **Combat-path performance budget.** Anything hooking `UnitScript::OnDamage`/`OnHeal` must have a bounded, *measured* per-call cost | These run at very high frequency; "it seemed fine" is not a measurement |
| T-8 | **Observability from day one.** Structured logging, GM inspection commands, and an audit trail for every grant, every currency movement and every life end | Required for debugging *and* exploit investigation |
| T-9 | **Idempotent grants.** Every acquisition path is idempotent and transactional | Otherwise duplicate rewards become an exploit class (X-8) |
| T-10 | **Stock-client expressibility.** Every player-facing affordance traces to a stock mechanism — gossip, spell, aura, item, action bar, chat, tooltip — at PLAN time, not ACT time | No MPQ patch, no custom DBC, no required AddOn |

---

## 3. Verified surface register

Cite this register instead of re-deriving. Anything not here is unverified until it is.
`PlayerScript` exposes **187** registered hooks in total.

### 3.1 Classless build construction (System A)

| Need | Verified surface |
|---|---|
| Suppress / repurpose talent points | `PlayerScript::OnPlayerCalculateTalentsPoints(Player const*, uint32& talentPointsForLevel)` — writable out-param |
| Gate talent learning | `OnPlayerCanLearnTalent`, `OnPlayerLearnTalents`, `OnPlayerTalentsReset`, `OnPlayerFreeTalentPointsChanged`, `OnPlayerBeforeInitTalentForLevel` |
| Grant / revoke abilities | `Player::learnSpell(uint32, bool temporary, bool learnFromSkill)`, `Player::addSpell(...)`, `Player::removeSpell(...)`, `Player::SetFreeTalentPoints(uint32)`, `Player::resetTalents(bool)` |
| Observe acquisition | `OnPlayerLearnSpell`, `OnPlayerForgotSpell`, `OnPlayerLevelChanged`, `OnPlayerFirstLogin`, `OnPlayerLogin`, `OnPlayerCreate`, `OnPlayerSave` |
| Starting kit and base stats (data-driven) | world DB `playercreateinfo`, `playercreateinfo_spell_custom`, `playercreateinfo_action`, `playercreateinfo_item`, `playercreateinfo_skills`, `player_class_stats` |

### 3.2 Roguelite run and life-end (System B)

| Need | Verified surface |
|---|---|
| Detect death | `OnPlayerJustDied`, `OnPlayerKilledByCreature`, `OnPlayerReleasedGhost` |
| **Enforce** hardcore (block revival) | `bool OnPlayerCanResurrect`, `bool OnPlayerCanRepopAtGraveyard` — boolean gates, so a module can genuinely prevent resurrection |
| Persist across lives | `OnPlayerDelete(ObjectGuid guid, uint32 accountId)`, `OnPlayerDeleteFromDB(CharacterDatabaseTransaction, uint32 guid)`, `AccountScript::OnAccountLogin`, `OnBeforeAccountDelete` |
| Level-milestone triggers | `OnPlayerLevelChanged`, `OnPlayerSetMaxLevel`, `OnPlayerCanGiveLevel` |
| Achievement-based triggers | `AchievementScript`, `AchievementCriteriaScript`, `OnPlayerAchievementSave`, `OnPlayerCriteriaSave` |

### 3.3 Balance, pacing and telemetry

| Need | Verified surface |
|---|---|
| Combat output modification | `UnitScript::OnDamage`, `OnHeal`, `OnAuraApply`, `OnAuraRemove`, `OnUnitDeath`, `OnUnitEnterCombat`, `OnUnitExitCombat`, `OnBeforeRollMeleeOutcomeAgainst` |
| Stat / scaling modification | `OnPlayerCustomScalingStatValueBefore`, `OnPlayerCustomScalingStatValue`, `OnPlayerAfterUpdateMaxPower` |
| XP pacing — multiplier | `FormulaScript::OnBaseGainCalculation`, `OnGainCalculation`, `OnGroupRateCalculation`; `OnPlayerGiveXP`, `OnPlayerQuestComputeXP`, `OnPlayerBeforeGetLevelForXPGain` |
| **XP pacing — curve shape** | world DB `player_xp_for_level` (`src/server/game/Globals/ObjectMgr.cpp:4928`) — XP required per level is fully data-driven |
| Level cap | config `MaxPlayerLevel` (`worldserver.conf.dist:2119-2125`, default 80, range 1–255); `DEFAULT_MAX_LEVEL 80`, `STRONG_MAX_LEVEL 255` (`src/server/shared/DataStores/DBCEnums.h:35,43`) |
| Loot / economy observation | `OnPlayerLootItem`, `OnPlayerAfterCreatureLoot`, `OnPlayerAfterCreatureLootMoney`, `OnPlayerMoneyChanged`, `OnPlayerCreateItem`, `OnPlayerQuestRewardItem` |
| Guardians / summons | `OnPlayerBeforeGuardianInitStatsForLevel`, `OnPlayerAfterGuardianInitStatsForLevel`, `OnPlayerBeforeTempSummonInitStats` |

**Consequence for pacing:** because the XP curve is data-driven, the ≈6h target is reachable by *reshaping
the curve per progression stage* rather than inflating `Rate.XP.*`. This satisfies the brief's constraint by
construction rather than by compromise.

---

## 4. Hard client constraints — immovable without a client patch

| Constraint | Evidence | Consequence |
|---|---|---|
| `[V]` Talent tabs are class-masked | `TalentTabEntry.ClassMask`, `src/server/shared/DataStores/DBCStructure.h:1946` | The stock talent UI **cannot** present a classless tree. Classless talents need a different delivery channel. |
| `[V]` Primary resource is class-bound | `ChrClassesEntry.powerType`, `src/server/shared/DataStores/DBCStructure.h:656` | A character has exactly one of mana/rage/energy/runic, fixed by its technical class. This is why a **chassis** exists — a client constraint, not a design preference. |
| `[V]` Character creation data is server-side | `playercreateinfo*`, `player_class_stats` | Starting kit and stat curves are ours to redefine in SQL |

---

## 5. Not yet verified — may not be relied upon

Flagged so no plan treats them as established.

- **V-1** Which level-60 raid and dungeon encounters exist in the 3.3.5a client, and at what tuning. The
  classic raids were re-tuned across expansions; their 3.3.5a state must be *measured*. Blocks endgame and
  solo-raid assessment (Phase 7).
- **V-2** Cross-chassis spell behaviour: scaling, stance/form requirements, resource assumptions, pet and
  aura assumptions, and which spells silently no-op. **Blocks the entire power catalogue** — hence Phase 1.
- **V-3** Whether classic 1–60 quest and dungeon content in the 3.3.5a world DB supports a coherent solo
  levelling path at the intended pace without gaps.
- **V-4** Behaviour of achievements, gear and reputations under a level-60 cap on a WotLK client.

---

## 6. Module hosting and layout

`[O]` **Q4 — unresolved.** Two options, both viable:

| Option | For | Against |
|---|---|---|
| **Separate repository**, cloned/submoduled under `modules/` | Standard AzerothCore practice; keeps upstream history clean; module versioned independently | Two repositories to keep in step for a solo maintainer |
| **Vendored in-tree** via a `!modules/mod-<name>` un-ignore | One repository, one history, simplest solo workflow | Diverges from AC convention; module cannot be shared or versioned separately |

`[V]` Relevant fact: `.gitignore:8-13` ignores `modules/*` except scaffolding, so either option requires a
deliberate act — the default does nothing.

`[P]` **Naming and namespacing** (pending Q4):
- Module directory `mod-<project-slug>`.
- C++ namespace matching the project slug; no global symbols.
- DB tables prefixed with the project slug; never a bare name that could collide with AzerothCore.
- Config keys under a single prefixed section.

---

## 7. Upstream rebase policy `[P]`

- Track `origin/master`.
- `[O]` Cadence to be set — proposal: rebase before each phase begins, so a phase never starts on a stale base.
- After each rebase, record the core diff against upstream in the phase's consolidation notes. A diff that
  grows without an accompanying ADR is a process failure.
- Core edits require: an ADR, a stated reason no hook suffices, and a rebase-risk note.

**Target:** the full core diff against upstream should stay small enough to read and explain in one sitting.

---

## 8. Performance budgets `[P]`

| Path | Budget | Verified by |
|---|---|---|
| `UnitScript::OnDamage` / `OnHeal` hooks | `[O]` bound to be set before the augment system ships (T-7) | Instrumented live run |
| Per-login persistent-state load | `[O]` | Instrumented live run |
| Telemetry write volume and retention | `[O]` TM-3 — stated growth budget required | Table growth measurement |

No numbers are invented here. Each is set from measurement during the phase that introduces the path.

---

## 9. Tooling: azerothMCP

`https://github.com/azerothcore/azerothMCP` — `[V]` official AzerothCore org, GPL-2.0, Python, active
(last push 2026-04-15, verified via `gh api`). Read-only MCP server over the world/characters/auth
databases with SmartAI, spell, quest, item, condition and waypoint tooling.

**Value:** three of the four unverified items in §5 are database-shaped. `search_spells` /
`lookup_spell_names` serve V-2 directly; `diagnose_quest` and creature search serve V-3 and V-1.

**Scheduled for immediately before Phase 1**, not earlier — it is a client over a *populated* MySQL, and
this project has no build and no database yet.

**Constraints when adopted:**
- Read-only inspection tool. It must never become a write path; schema change continues through
  `pending_db_*/` (T-5).
- It does not relax T-2 — facts it reports are cited with the query that produced them.
- `ENABLE_SOURCE_CODE` and `ENABLE_WIKI` stay off unless a specific need arises; both cost tokens.
- Adoption is recorded as an ADR since it becomes part of the verification workflow.

---

## 10. Open questions affecting this document

`Q3` power source (existing spells cross-chassis / original effects / mix — determines whether this is a
curation or a C++ effects project) · `Q4` module hosting · `Q7` nerf policy · `Q10` planning-doc location.
Full list: requirements §14.
