# Project Vision

**Status:** Proposed — awaiting owner approval. Nothing here is a final decision.
**Last updated:** 2026-09-03
**Source:** `requirements/project-foundation.REQUIREMENTS.md` (v2)
**Legend:** `[V]` verified in-repo · `[A]` assumption · `[P]` proposal · `[O]` open question

---

## 1. What this is

An original, non-commercial AzerothCore 3.3.5a private server in which a player's class does not exist.
A **life** is a journey from level 1 to level 60. During that life the player assembles a classless build
from curated cross-class abilities, talent-like passives and run-scoped roguelite augments. When the life
ends, the build is gone; what carries forward is account-level progression that makes the next life wider,
stranger and faster.

The classic 1–60 world is not a backdrop to the systems. It **is** the run.

## 2. Player fantasy

> *"I have played this world before. I have never played it as this."*

The player is not levelling a warrior or a priest. They are levelling a **hypothesis** — a build that did
not exist until this life, discovered through what they actually chose to do in the world, tested against
content they recognise but cannot approach the same way twice.

The satisfaction loop is not "my character got stronger". It is **"I found something that works, and I
found it myself."**

## 3. Why play this rather than the alternatives

This section exists because of Risk R-1: the project reuses stock content, so the systems alone carry the
identity. If this section is not convincing, the project does not have a reason to exist yet. It should be
re-read and challenged before every major phase.

| Alternative | What it offers | What it does not |
|---|---|---|
| Retail / Classic WoW | A polished world and fixed, well-tuned class identities | Your build is stable for months; progression is gear acquisition, not construction. Replaying the levelling journey is the same journey. |
| Other classless servers | Freedom to mix abilities across classes | Usually one permanent character. Players converge on a settled build and stop building. The classless-ness is a character-creation feature, not an ongoing activity. |
| Run-based roguelite servers | Genuine run structure and escalating power | Usually instanced or arena-shaped. The run is disconnected from the world journey, so the world stops mattering. |

**Our claim** — the combination none of the above make:

1. **The world journey is the roguelite run.** Questing, exploration, elites, gear and dungeons matter
   because they are how you acquire your build, not merely how you gain levels.
2. **Building is the skill you get better at.** Across lives the player accumulates knowledge of synergies
   and, through persistent progression, the *agency* to act on it.
3. **Repetition is the point, and it is not repetitive.** The same zones, a different set of tools, every
   life.

**The honest failure mode.** If the power system is shallow, this is "talent trees with extra steps" and
none of the above is true. The Design Pillars exist specifically to make that failure detectable early.

## 4. Audience

- `[A]` Players who enjoy building and system mastery more than raiding schedules.
- `[A]` Players comfortable with a Vanilla-style pace, at least for the first life.
- `[A]` Small-group PvE players — the endgame baseline is three people, not twenty-five.
- `[A]` Solo players — the first life is solo-friendly by requirement, and late progression supports
  deliberate solo raid attempts.
- Scale and openness of the realm is **[O] Q5**, and it materially changes the anti-exploit budget.

## 5. Scope boundaries

- WoW 3.3.5a client, **unmodified**. No MPQ patch, no custom DBC, no required AddOn. Every player-facing
  affordance must be expressible through stock gossip menus, spells, auras, items, action bars and tooltips.
- **PvE only. PvP is permanently out of scope** and is not a balance dimension.
- Level cap 60. `[V]` This is a configuration value (`MaxPlayerLevel`, range 1–255,
  `worldserver.conf.dist:2119-2125`), not a code change.

## 6. What success looks like

No targets are set yet — we have no data, and inventing numbers now would be exactly the guesswork the
brief forbids. These are the metrics that will carry targets, and the reason each was chosen.

| # | Signal | Why this one | Target |
|---|---|---|---|
| SM-1 | First-life completion rate (reached 60) | If players do not finish one life, nothing downstream matters | `[O]` |
| SM-2 | **Second-life start rate** | The single most important roguelite signal. If players finish a life and never begin another, the entire premise is wrong | `[O]` |
| SM-3 | Median first-life 1→60 time | The brief's ≈6h target; also the clearest pacing regression detector | ≈6h `[P]` |
| SM-4 | Build diversity — pick-rate ceiling for any ability, talent or augment | Detects the mandatory-best-build failure (X-11) | `[O]` |
| SM-5 | Role coverage — share of lives reaching 60 as tank / healer / DPS / hybrid | Detects a role that is nominally supported but practically unviable | `[O]` |
| SM-6 | 3-player endgame participation | The stated endgame baseline; if unused, the endgame design is wrong | `[O]` |
| SM-7 | Anti-exploit register violations reaching production | Must be zero; each row of the register has a negative test | 0 |
| SM-8 | Players can articulate why their build works | Legibility is a retention hypothesis (H-5); measured qualitatively | `[O]` |

Metric plumbing is specified in `BALANCE_FRAMEWORK.md` §4 and is a Phase 3 prerequisite.

## 7. Legal and commercial stance

- **Original work only.** No third-party server's code, database, UI, assets, branding, terminology, exact
  mechanics, balance values, custom content, text or proprietary designs are used. Where other projects are
  referenced, it is for publicly-observable *genre principles* only, and the implementation here is
  independently designed. (Requirements NG-8.)
- **Non-commercial.** No monetisation, no paid access, no donation-for-power, no commercial activity of any
  kind. (Requirements NG-9.)
- **No Blizzard asset creation or redistribution.** The project operates against a client the operator
  already owns; it produces no client files, game data, art, music or branding.

## 8. Open questions affecting this document

`Q1` life = character or reset-in-place · `Q2` acquisition model · `Q3` power source ·
`Q5` audience scale · `Q6` chassis visibility · `Q8` persistent currency shape.
Full list: requirements §14.
