# Core Game Loop

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-03
**Governed by:** D-001r (persistent-progression roguelite with per-life classless builds) and
D-002 (voluntary prestige, plus opt-in stakes). See `DECISION_LOG.md`.

---

## 1. Loop hierarchy

```
L5  PERSISTENT LOOP        across lives ......... unlock agency -> start a wider life
     |
L4  LIFE LOOP              ~6h first, faster later
     |                     level 1 -> 60 -> endgame -> prestige (voluntary)
     |
L3  ENDGAME LOOP           at 60, open-ended
     |                     3-player content / solo raid attempts / prepare to prestige
     |
L2  SESSION LOOP           one sitting
     |                     pick a goal -> pursue it -> acquire -> re-evaluate build
     |
L1  CORE LOOP              moment to moment
                           fight -> acquire -> adapt
```

Each loop must be satisfying **on its own**, without the loop above it. A player who plays one session and
stops should have had a good time; a player who plays one life and never prestiges should still have had a
complete experience.

---

## 2. L1 — Core loop (seconds to minutes)

**Fight → Acquire → Adapt.**

1. Engage world content with the kit you currently have.
2. Levelling, quest completion, exploration and defined PvE achievements yield acquisition events —
   abilities, talents, or (at milestones) augments.
3. The new piece changes how you fight, so the next fight is approached differently.

**What makes this good:** the acquisition is *legible* (Pillar 4) and *chosen under constraint*
(Pillar 2). The player immediately understands what changed and what they gave up.

**Failure mode:** acquisition that does not change how the next fight plays. A grant the player cannot feel
is a wasted acquisition event, and acquisition events are the project's scarcest design resource.

---

## 3. L2 — Session loop (one sitting)

**Pick a goal → pursue it in the world → acquire → re-evaluate the build.**

A session is bounded by a world goal — a zone, a quest chain, a dungeon, an elite. The player leaves the
session with a build meaningfully different from the one they arrived with.

- `[P]` A session should contain **at least one acquisition decision the player thinks about**. If a
  two-hour session contains no real choice, the session loop is flat.
- `[P]` The player should be able to stop at any point without losing run progress. Run state is persistent
  within the life; only *ending the life* resets it.

**Open:** session length assumptions depend on Q5 (audience scale) and the final pacing curve.

---

## 4. L3 — Endgame loop (at 60, open-ended)

Under D-002, **level 60 is a home, not a finish line.** The player may stay indefinitely.

- **Default target:** a 3-player group — one tank, one healer, one DPS (P-15). Multiple distinct archetypes
  must be capable of each role (P-5).
- **Solo track:** at late persistent progression, specialised builds may solo *selected* raid encounters.
  Each boss is individually assessed and the assessment recorded (P-17).
- **Multi-player-only mechanics** get deliberate, documented adaptations. An encounter that becomes
  soloable through an unintended bypass is a **defect** (P-18).
- Group play must remain rewarding even though high-progression solo play exists (P-16).

**Unverified and blocking:** `[A]` V-1 — which level-60 encounters exist in the 3.3.5a client and at what
tuning. The classic raids were re-tuned across expansions. Nothing in this section may be treated as
designed until V-1 is measured (Phase 7).

---

## 5. L4 — Life loop (level 1 → 60 → prestige)

### 5.1 Life start

- The **vessel** carries a **chassis** (technical class, `[V]` required because `ChrClassesEntry.powerType`
  binds resource type — `DBCStructure.h:656`). Per D-007 the vessel persists across lives, so the chassis is
  chosen at vessel creation, not at each life start, **unless prestige may change it** — `[O]` Q11.
  Visibility to the player is `[O]` Q6.
- The player receives a small **viable** starting kit (P-2). A dead starting kit is a defect (X-1).
- The player may declare **optional stake modifiers** — hardcore, or challenge conditions — which are
  visible for the life's duration and **immutable once chosen** (P-12). `[Amended D-030]` The **difficulty
  tier** is *not* one of these: it is a modifier layer over the world, unlocked by content and switchable
  while rested at an inn or capital. Only the death-ends-life stake is immutable.
- `[P]` Persistent unlocks apply here: better starting *options*, wider drafts, rerolls, curated pools.
  This is where "agency before power" (Pillar 3) is most visible to the player.

### 5.2 Life body

Levelling 1→60 through the classic world. Three power layers accumulate, and they are never conflated:

| Layer | Nature | Acquired |
|---|---|---|
| **Abilities** | active tools | through levelling, by guaranteed-offer draft (D-008) |
| **Talents** | passive modifiers, specialisation | through levelling, by guaranteed-offer draft (D-008) |
| **Augments** | build-warping interactions — resource, cooldown, trigger, area, scaling transformation, defensive conversion, risk/reward | level milestones, and potentially defined PvE achievements |

**Banking (D-029).** Currency accrues during a life and is credited to the account **only at life end**,
scaled by what the life achieved — distinct dungeons and raids completed, achievements earned, level reached,
and *active* played time. So level 60 is genuinely a home: you live there, run content, grow the bank, and
cash out when you choose. Both life-end paths use the same formula.

**Draft tools (D-027).** A life carries three counted, account-earned tools: **reroll** replaces the whole
offer, **hold** carries one option into the next offer, **remove** takes an option out of this life's
eligible pool for good. Remove is a player-authored eligibility rule, so it improves every later offer
rather than just the current one — and its budget is a *variance* dial, not a convenience one (D27-d).

**Cadence (D-009).** A life opens with a **wild card** ability or talent selection. Then **at every level**
the player chooses one of three: a new ability, an **upgrade** to an ability they already hold, or a talent.
The upgrade is a fourth axis — *depth* rather than a fourth power layer — and it competes for the same slot
as breadth, which is where Pillar 2's real cost bites. Augments keep their own milestone rhythm and are not
part of the per-level choice.

Pacing: ≈6h for an informed ordinary player on a first life (P-8); progressively faster on later lives
through unlocks, agency and synergy (P-10) — **not** primarily through an XP multiplier.
`[V]` This is achievable by reshaping the data-driven XP curve in `player_xp_for_level`
(`ObjectMgr.cpp:4928`) rather than inflating `Rate.XP.*`.

### 5.3 Life end

Two paths, per D-002:

| Path | Trigger | Notes |
|---|---|---|
| **Prestige (default)** | Player's deliberate, confirmed choice at 60 | Consequences shown before commitment (P-11). Death during a default life is ordinary WoW death with no life consequence. |
| **Staked end** | Declared stake condition met — e.g. death in a hardcore life | The stake multiplied earnings *throughout* the life; the ending itself pays **no** bonus, and uses the **same** payout formula as prestige (D-029, X-15). Must be fully auditable: cause, location, source, timestamp (D2-b). |

**Constraint D2-a:** no persistent unlock may be reachable *only* through a staked life. If it is, "optional"
is a false description and the model collapses into mandatory hardcore.

**Settled by D-007 (2026-09-04).** A life is **not** a character. The character is a persistent **vessel**;
prestige resets that vessel in place to level 1, and each life is recorded as a first-class row carrying its
stake modifiers, timestamps, outcome and audit trail. Consequences that land in this section:

- The 10-character client cap (`worldserver.conf.dist:2031-2036`) bounds **concurrent vessels**, not lives.
  A player may run several vessels under different stakes.
- Past lives remain queryable, so a "past lives" view is possible. Not committed to here.
- `[O]` The chassis question this raises is Q11, flagged in §5.1 above.
- The prestige reset itself is the project's highest-risk operation (D7-a) and needs an enumerated
  table-by-table reset manifest before implementation. That is a `QA_STRATEGY.md` gate, not a loop concern.

Still open in this section: `[O]` Q9 — whether disconnect and server-fault deaths are forgiven, appealed, or final in a
staked life. Silence here is a player-trust failure.

---

## 6. L5 — Persistent loop (across lives)

**End a life → convert it into permanent agency → start a wider life.**

- Rewards favour **agency before power** (P-13, Pillar 3): starting options, draft width, curated pools,
  rerolls, build slots, controlled and capped progression modifiers.
- Any raw-power reward requires caps, pacing, diminishing returns and telemetry *before it ships*.
- Persistent progression is `[A]` account-scoped and non-tradeable — this is also the primary defence
  against boosting and trading exploits (X-10).
- A returning player must be able to see what they have unlocked and what it changes (P-14).

**Open:** `[O]` Q8 — one currency, several, or direct unlock-on-achievement with no currency at all.

**Hypothesis under test:** H-3 — agency rewards produce most of the later-life speed gain. If falsified,
the persistent economy is redesigned; it is not patched with raw power.

---

## 7. Designed tensions

These conflicts are intentional and must be resolved explicitly in ADRs, never silently in code.

| Tension | Between | Resolution stance |
|---|---|---|
| World matters vs later lives are faster | Pillar 1 vs P-10 | Speed comes from agency and synergy, not from skipping content |
| 60 is a home vs prestige should happen | L3 vs L5 | Prestige must be *pulled* by desirable unlocks, never *pushed* by making 60 unpleasant |
| Solo viability vs 3-player baseline | P-16 vs P-17 | Group play stays rewarding; solo raid capability is a late, assessed, per-boss privilege |
| Build freedom vs no mandatory best build | Pillar 2 vs X-11 | Constraint budgets and eligibility rules, monitored by pick-rate ceilings (SM-4) |
| Legibility vs stock-client limits | Pillar 4 vs T-10 | Every affordance traced to a stock mechanism at PLAN time; complexity that cannot be communicated is cut |

---

## 8. Open questions affecting this document

`Q1` (blocking §5) · `Q2` acquisition model · `Q5` audience scale · `Q6` chassis visibility ·
`Q8` currency shape · `Q9` staked disconnect policy. Full list: requirements §14.
