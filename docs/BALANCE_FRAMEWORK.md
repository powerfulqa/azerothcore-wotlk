# Balance Framework

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-03
**Governing pillar:** Pillar 5 — Evidence over intuition. No balance change ships without a metric that
could have proven it wrong.

---

## 1. Balance dimensions

Every ability, talent, augment and persistent unlock is assessed against these. A proposal that does not
state which dimensions it moves is not ready for review.

Damage · survivability · healing · threat and mitigation · utility · mobility · crowd control ·
resource economy · level and gear scaling · **acquisition effort** · skill ceiling · group value ·
exploit potential.

**PvP is excluded permanently** (Requirements NG-7). It is not a dimension, not a tiebreaker, and not a
reason to reject a design.

**Acquisition effort is a first-class lever** (H-2): a harder-to-obtain power may be stronger, *provided*
acquisition is not repeatably farmable. This is what lets us ship spikes of power without power creep.

---

## 2. Target performance bands

Bands are specified across six axes. The sixth is unique to this project and is the one most likely to be
forgotten in a review.

1. Content tier
2. Role — tank / healer / DPS / hybrid
3. Group size — solo vs 3-player
4. Encounter duration
5. Gear level
6. **Persistent-progression stage** — a build at persistent-stage 1 and one at stage 10 are different games

**No numeric bands are set yet.** Setting them now would be the guesswork the brief forbids. They are
derived from telemetry (§4) once Phase 3 lands, and recorded here with the query that produced them.

`[O]` Open: how many persistent stages exist, and whether bands are defined per stage or as a curve —
depends on Q8.

---

## 3. Tuning method

1. **State the hypothesis.** What should change, and which metric would falsify it.
2. **Confirm the metric already exists.** If it does not, instrument first. A change shipped blind cannot
   be evaluated and cannot be safely reverted.
3. **Change data, not code** (Requirements T-4). Tunables live in module config and custom DB tables so
   that tuning needs no rebuild.
4. **Change one thing.** Simultaneous changes to interacting systems produce uninterpretable data.
5. **Observe against the band**, then record the outcome here — including changes that did not work.
6. **Revert cleanly.** Every tuning change states its rollback.

**Anti-pattern to refuse:** tuning a number because it felt strong in one session. That is a reason to
measure, not to change.

---

## 4. Telemetry requirements

Telemetry is a **prerequisite** for balance, not an afterthought — hence Phase 3 sits before the systems it
judges. Each metric names the layer that must produce it.

| # | Metric | Produced by |
|---|---|---|
| M-1 | First-life and repeat-life 1→60 time (wall-clock and played) | life record |
| M-2 | Ability pick rates | acquisition ledger — must record **options offered and not taken**, or this measures availability, not preference (D-008) |
| M-3 | Talent pick rates | acquisition ledger |
| M-4 | Augment pick rates | acquisition ledger |
| M-5 | Build distribution and extreme outliers | build snapshot at milestones |
| M-6 | Damage, healing, mitigation, threat, resource performance | combat aggregation |
| M-7 | Deaths, wipes, and life-end causes | death / life-end audit (D2-b) |
| M-8 | Dungeon, raid and solo-raid completion rates | encounter records |
| M-9 | Currency generation and spending | economy ledger |
| M-10 | Reward acquisition | acquisition ledger |
| M-11 | Exploit indicators (§5) | anti-exploit counters |
| M-12 | Per-life category coverage at life end — which role-functions the build ended up with — correlated against completion and quit behaviour (D21-e) | acquisition ledger + life record |

**Constraints:**

- **TM-1** Metrics must be queryable offline in SQL, without attaching a debugger to a live server.
- **TM-2** Build snapshots must be reconstructable after the fact, or M-2…M-5 are unanalysable. Recording
  only the *final* build loses every rejected choice, which is where the interesting data lives.
- **TM-3** Telemetry must have a stated retention and growth budget. A row per damage event is not viable
  on a live realm and must not be proposed casually. `[O]` The aggregation granularity for M-6 is open and
  should be settled during Phase 3, not during the first system that needs it.

**Verified surface available for collection** (see `TECHNICAL_ARCHITECTURE.md` §3 for the full register):
`OnPlayerLevelChanged`, `OnPlayerLearnSpell`, `OnPlayerLootItem`, `OnPlayerMoneyChanged`, `OnPlayerJustDied`,
`OnPlayerKilledByCreature`, `UnitScript::OnDamage` / `OnHeal` / `OnUnitDeath`, and the achievement hooks.

---

## 5. Anti-exploit register

Every item the brief lists as unacceptable, with the layer that prevents it. **Each row requires at least
one negative test** in `e2e/suites/` before the system that could cause it ships (see `QA_STRATEGY.md`).

| # | Must not occur | Primary prevention |
|---|---|---|
| X-1 | Dead starting kits | Requirement P-2 as an acceptance criterion; automated viability check over every legal starting kit |
| X-2 | Unusable ability rolls | Tag and eligibility rules (P-4) plus the Q2 acquisition model |
| X-3 | Infinite resource loops | Augment interaction rules; resource-generation caps; combat-log assertions |
| X-4 | Recursive damage or healing loops | Trigger-depth limit and re-entrancy guard designed into the augment trigger system from the start |
| X-5 | Permanent invulnerability | Mitigation stacking caps; duration floors; explicit uptime assertions |
| X-6 | Infinite crowd control | CC uptime caps; diminishing returns preserved for PvE |
| X-7 | Pet duplication | Guardian/summon hooks (`OnPlayerBeforeGuardianInitStatsForLevel`) plus summon count caps |
| X-8 | Reward duplication | Idempotent, transactional grants (T-9) plus acquisition audit trail |
| X-9 | Reset abuse | Life-end and prestige rules; prestige eligibility constraints; M-7 monitoring |
| X-10 | Trading, boosting and economy exploits | Persistent progression is account-scoped and non-tradeable; group-acquisition rules; M-9 monitoring |
| X-11 | A single mandatory best build | Role coverage (P-5); outlier detection via M-2…M-5; pick-rate ceiling SM-4; a stated nerf policy |
| X-12 | Draft rerolling by relog or session manipulation | Offers generated once and persisted transactionally with the acquisition point (D8-a); reroll spend recorded in the life row |
| X-13 | A drafted proficiency whose gear never drops — a wasted slot, not an exploit but the same felt cost | Measure drop and quest-reward distribution by armour/weapon type and level band (V-3) before finalising the draft pool; keep the baseline set wide enough that no life is stranded (D20-c) |
| X-14 | Loadout swapping to trivialise encounters — slotting tank tools for one fight, then swapping back | Replacement occurs **only at acquisition**; there is no free re-slotting (D23-a). Any future convenience that relaxes this reopens the exploit |

**X-3 and X-4 are architectural, not incidental.** Trigger-depth guards must exist in the augment system's
first commit. Retrofitting a re-entrancy guard into a shipped trigger system is how servers get duplication
bugs.

**Open policy gap — `[O]` Q7.** X-11 needs a decided answer to: *what happens to a player who already owns a
power we nerf?* Retroactive, grandfathered, or refunded. Until Q7 is answered we cannot honestly promise to
fix an outlier build after it ships.

---

## 6. Hypotheses under test

Stated so they can be falsified with data rather than defended by argument. Each should be revisited when
the phase that can test it completes.

| # | Hypothesis | Falsified by | Testable from |
|---|---|---|---|
| H-1 | Tags and eligibility rules alone prevent dead builds, with no manual per-combination blocklist | Dead builds appearing despite valid tags | Phase 4 |
| H-2 | Acquisition effort is a real balance lever if acquisition is not farmable | Strong hard-to-get powers still distorting M-2…M-5 | Phase 4 |
| H-3 | Persistent **agency** rewards produce most of the later-life speed gain | M-1 not improving across lives without raw-power unlocks | Phase 3 + 6 |
| H-4 | The dominant exploit surface is acquisition-rate and reset abuse, not raw power | M-11 showing power-based exploits dominating | Phase 5 |
| H-5 | Legibility matters more for retention than depth | SM-2 not correlating with SM-8 | Post-launch |
| H-6 | Stock 3.3.5a level-60 content will be trivialised by an optimised build; scaling or retuning will be required | Content proving appropriately tuned as-is | Phase 7, after V-1 |
| H-7 | A 3-player tank/healer/DPS target is achievable without per-encounter bespoke tuning | Encounters requiring individual adaptation to be completable | Phase 7 |
| H-8 | An unweighted combined-pool draw does not starve early lives of abilities, because rerolls and injected guarantees absorb bad offers (D22-a) | M-2/M-12 showing levels 2–8 offers skewed to proficiency, or early-life abandonment correlating with low early ability count | Phase 4 |

H-7 is the most optimistic entry here and should be tested early rather than assumed through to launch.

---

## 7. Guardrails against power creep

- Persistent raw power is capped, paced and subject to diminishing returns (P-13).
- Every power ships with its pick rate instrumented from day one (Pillar 5).
- A pick-rate ceiling breach (SM-4) triggers investigation, not an automatic nerf — a popular power may be
  popular because it is *legible* (H-5), which is a success, not a fault.
- Build diversity and role coverage (SM-4, SM-5) are release-blocking metrics, not nice-to-haves.
