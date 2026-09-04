# Design Pillars

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-03
**Purpose:** These five pillars are the accept/reject test for **every** future feature. A proposal that
fails a pillar is rejected or reworked, regardless of how good it sounds in isolation. If a pillar starts
blocking things we clearly want, the pillar is wrong and gets an ADR — it does not get quietly ignored.

---

## Pillar 1 — The world is the run

The classic 1–60 journey is the delivery vehicle, not scenery. Systems must make questing, exploration,
elites, gear and dungeons matter *more*, never less.

**Test:** *Does this make a player engage with the world more deliberately, or does it let them skip it?*

| Accept | Reject |
|---|---|
| An augment earned by clearing a specific dungeon | A vendor that sells the same augment in a hub |
| Acquisition tied to exploring a zone | Acquisition from a menu at level 1 |
| Faster later lives via better choices | Faster later lives via a large XP multiplier alone |

**Guards:** Requirements P-9, P-10. Watch for this pillar eroding gradually — every individual convenience
feature is defensible; the sum of them deletes the world.

---

## Pillar 2 — Build, don't pick

Power is **constructed** from discovered parts under constraint. There is no settled build and no single
correct answer. **ADR-0011 briefly qualified this and was superseded by ADR-0012:** every chassis carries
every resource, so the chassis does not bound the ability pool and the pillar stands unqualified. The
chassis fixes only the displayed resource, base stats and armour animation.

**Test:** *Is the player making a decision with a real cost, or collecting an item from a list?*

| Accept | Reject |
|---|---|
| A choice between three options with different roles | A grant with no alternative forgone |
| Constraints that make two strong powers mutually exclusive | Accumulation until every slot is filled with the best thing |
| Several viable archetypes per role | One optimal build per role |

**Guards:** Requirements P-3, P-4, P-5, X-11. Metric: SM-4 pick-rate ceiling.

**Note:** This pillar is why the acquisition model was the largest open design question. It is settled by
**ADR-0008** — a choice-of-N draft in which protected categories guarantee a role-critical function is
*offered* rather than possessed. The "offered, not possessed" framing exists precisely to satisfy this
pillar: a guarantee that handed the player the power would be a grant with no alternative forgone, which
this pillar rejects.

---

## Pillar 3 — Agency before power

Persistent progression **widens choices before it raises numbers**. Raw power is the last resort, always
capped, paced and instrumented.

**Test:** *Does this unlock give the player a new decision, or just a bigger number?*

| Accept | Reject |
|---|---|
| An extra draft option; a reroll; a wider curated pool; a build slot | A flat +10% damage account-wide |
| A better starting kit *choice* | A strictly better starting kit |
| A capped modifier with diminishing returns and telemetry | An uncapped stacking bonus |

**Guards:** Requirements P-13, H-3. If H-3 is falsified — if agency rewards do *not* produce the later-life
speed gain — the persistent economy needs redesign, not a power injection.

---

## Pillar 4 — Legible strangeness

Builds may be wild. They must remain **explainable**. A player should be able to say, in one sentence, why
their build works. Power the player cannot understand is churn, not depth.

**Test:** *Could a player who just acquired this explain what it does and why they took it?*

| Accept | Reject |
|---|---|
| An augment that visibly transforms a known ability | A hidden multiplier buried in a stat |
| Synergies discoverable from tooltips and observed behaviour | Synergies that require a spreadsheet or an external wiki |
| Clear feedback when a trigger fires | Silent internal state the player must infer |

**Strained by ADR-0024**, which left the augment layer uncapped. Fifteen stacked warping effects is not
obviously explainable in one sentence. The mitigation is the ADR-0013 AddOn: it **must** surface augment
state and active interactions, and **H-9** exists to measure whether that is enough. If H-9 is falsified,
this pillar and ADR-0024 are in genuine conflict and one of them must change by ADR.

**Guards:** Requirements P-6, P-7, H-5, T-10. This pillar was under constant pressure from the stock-client
constraint, which made it harder here than on a server with an AddOn. **ADR-0013 lifts that handicap:** the
project now ships its own bundled AddOn, so custom UI is available. The pillar still binds — a rich UI makes
it easier to *hide* complexity behind a readout rather than design it away — but it is no longer fighting the
client.

---

## Pillar 5 — Evidence over intuition

No balance change ships without a metric that could have proven it wrong. "It feels strong" is a reason to
*measure*, not a reason to change a number.

**Test:** *Which metric would tell us this change was a mistake, and is that metric already collected?*

| Accept | Reject |
|---|---|
| A tuning change with a named metric and a target band | A nerf because it seemed strong in one session |
| A power shipped with its pick-rate tracked from day one | A power shipped with no instrumentation |
| A stated hypothesis that can be falsified | An argument that cannot be settled by data |

**Guards:** Requirements §7 (telemetry), §8 (framework), and the evidence rule in `QA_STRATEGY.md`. This is
why the telemetry substrate is Phase 3 — before the systems it exists to judge.

---

## Using the pillars

- Every requirements document names the pillars its feature serves and any it strains.
- A feature that fails a pillar goes back to REFINE, not to implementation with a caveat.
- Pillar conflicts are real and expected — Pillar 1 (world matters) and pacing (P-10, faster later lives)
  pull against each other by design. Resolve conflicts explicitly in an ADR; do not resolve them silently
  in code.
