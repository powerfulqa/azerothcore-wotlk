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

- The player selects a **chassis** (technical class, `[V]` required because `ChrClassesEntry.powerType`
  binds resource type — `DBCStructure.h:656`). Visibility to the player is `[O]` Q6.
- The player receives a small **viable** starting kit (P-2). A dead starting kit is a defect (X-1).
- The player may declare **optional stake modifiers** — hardcore, or challenge conditions — which are
  visible for the life's duration and **immutable once chosen** (P-12).
- `[P]` Persistent unlocks apply here: better starting *options*, wider drafts, rerolls, curated pools.
  This is where "agency before power" (Pillar 3) is most visible to the player.

### 5.2 Life body

Levelling 1→60 through the classic world. Three power layers accumulate, and they are never conflated:

| Layer | Nature | Acquired |
|---|---|---|
| **Abilities** | active tools | through levelling, model `[O]` Q2 |
| **Talents** | passive modifiers, specialisation | through levelling, model `[O]` Q2 |
| **Augments** | build-warping interactions — resource, cooldown, trigger, area, scaling transformation, defensive conversion, risk/reward | level milestones, and potentially defined PvE achievements |

Pacing: ≈6h for an informed ordinary player on a first life (P-8); progressively faster on later lives
through unlocks, agency and synergy (P-10) — **not** primarily through an XP multiplier.
`[V]` This is achievable by reshaping the data-driven XP curve in `player_xp_for_level`
(`ObjectMgr.cpp:4928`) rather than inflating `Rate.XP.*`.

### 5.3 Life end

Two paths, per D-002:

| Path | Trigger | Notes |
|---|---|---|
| **Prestige (default)** | Player's deliberate, confirmed choice at 60 | Consequences shown before commitment (P-11). Death during a default life is ordinary WoW death with no life consequence. |
| **Staked end** | Declared stake condition met — e.g. death in a hardcore life | Grants additional persistent reward proportionate to risk. Must be fully auditable: cause, location, source, timestamp (D2-b). |

**Constraint D2-a:** no persistent unlock may be reachable *only* through a staked life. If it is, "optional"
is a false description and the model collapses into mandatory hardcore.

**Open and unanswered:** `[O]` **Q1 — is a life a character, or a character reset in place?** This is the
next structural decision. It determines whether prestige retires/deletes the level-60 character and starts a
fresh one, or resets the same character to level 1 — and with it the persistence anchor, character-slot
pressure, and whether past lives remain viewable. **Sections 5.1 and 5.3 cannot be finalised until Q1 is
answered.**

Also open: `[O]` Q9 — whether disconnect and server-fault deaths are forgiven, appealed, or final in a
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
