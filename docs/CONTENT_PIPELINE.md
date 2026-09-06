# Content Pipeline

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-03
**Purpose:** How a single piece of content goes from idea to live, and the gates it passes through. The
pipeline exists so that content can be produced *in volume* without each item being a bespoke risk.

---

## 1. Content types

These are distinct and must never be conflated in design, data, UI or conversation.

| Type | Nature | Lifetime | Layer |
|---|---|---|---|
| **Ability** | Active tool the player casts or uses | Current life | System A |
| **Talent** | Passive modifier or specialisation | Current life | System A |
| **Augment** | Build-warping interaction — resource, cooldown, trigger, area, scaling transformation, defensive conversion, risk/reward | Current life | System B |
| **Persistent unlock** | Account-level agency: starting options, draft width, curated pools, rerolls, build slots, capped modifiers | Across lives | System B |
| **Encounter adaptation** | Documented change to a multi-player-only mechanic so it behaves deliberately for smaller groups | Permanent | Endgame |

**Rule:** an item's type is decided before it is designed. "It could be a talent or an augment" means the
design is not finished.

---

## 2. Originality gate — applies to every item

Before anything else: the item is **original**. No third-party server's mechanics, terminology, balance
values, text or designs are reproduced (Requirements NG-8). Genre-level inspiration is permitted; specific
implementations are not. An item whose description could only have been written by looking at another
project's content is rejected.

---

## 3. Pipeline stages

```
1 CONCEPT        -> 2 CONTENT RECORD -> 3 TAGS/ELIGIBILITY -> 4 DATA AUTHORING
                                                                    |
8 OBSERVE        <- 7 RELEASE        <- 6 VERIFICATION     <- 5 IMPLEMENTATION
```

### Stage 1 — Concept
A one-paragraph statement of the player experience. Which pillar does it serve? What decision does it
create? What does the player give up to take it?

**Gate:** fails Pillar 2 (creates no real choice) or Pillar 4 (cannot be explained in a sentence) → rejected here, cheaply.

### Stage 2 — Content record
Every item gets a record before it gets data. Minimum fields in §4.

**Gate:** balance dimensions named (`BALANCE_FRAMEWORK.md` §1). An item whose dimensions are unstated cannot be reviewed.

### Stage 3 — Tags and eligibility
Tags assigned; eligibility rules stated; incompatible or exploit-prone combinations excluded **by rule, not
by blocklist** (H-1). Interaction with existing items considered — particularly trigger-based augments
against X-3 and X-4.

**Gate:** an item requiring a bespoke per-combination exception is a signal the tag system is wrong. Fix the
system, do not add the exception.

### Stage 4 — Data authoring
Values into module config and custom DB tables (T-4). Migration lands in `pending_db_*/` with its rollback
stated (T-5).

**Gate:** no tunable value hard-coded in C++. No exceptions — this is what makes Pillar 5 possible.

### Stage 5 — Implementation
Only if the item needs code beyond data. Module-first (T-1). Any core edit requires an ADR.

**Gate:** verified surface only (T-2, `TECHNICAL_ARCHITECTURE.md` §3). No invented hooks or columns.

### Stage 6 — Verification
Per `QA_STRATEGY.md` §3, by change type. Anti-exploit negative tests for any register row it could touch.

**Gate:** the evidence rule. Output shown, or it did not pass.

### Stage 7 — Release
Instrumentation live **before** the item is (Pillar 5): pick rate at minimum (M-2/M-3/M-4), plus whatever
dimension-specific metric its record named.

**Gate:** an item cannot ship without the metric that would later justify changing it.

### Stage 8 — Observation
Pick rate against the ceiling (SM-4); role coverage impact (SM-5); outlier detection (M-5); exploit
counters (M-11). Outcome recorded — including "no change needed".

**Gate:** a pick-rate breach triggers *investigation*, not an automatic nerf. A popular item may be popular
because it is legible, which is a success.

---

## 4. Content record — required fields

```
id / slug
type              ability | talent | augment | persistent unlock | encounter adaptation
concept           one paragraph: the player experience
pillars served    and any strained
decision created  what the player gives up to take this
tags
eligibility       rules; what it cannot combine with, and why
balance dimensions moved
acquisition       where it comes from; is it farmable? (H-2)
metrics           which metric would show this was a mistake
exploit review    which register rows it could touch (X-1..X-11)
verification      which QA gates apply
rollback          how it is removed if wrong
```

An item missing `metrics` or `rollback` is not ready. Those two are the ones most often skipped and most
expensive to add later.

---

## 5. Working in batches

Content is authored in **themed batches**, not one item at a time:

- A batch shares a review pass, a migration and a verification run.
- A batch is designed for *coverage*: does it add options for every role (P-5), or only for the popular one?
- A batch must not contain two items that only make sense together — that is one item wearing two names.
- Batch size is bounded by what can be verified in one pass. An unverifiable batch is too big.

---

## 6. Encounter adaptations

Distinct from item content because the failure mode is different.

1. **Assess** the encounter — is it soloable, 3-player viable, or neither? Record the assessment (P-17).
2. **Identify** multi-player-only mechanics explicitly.
3. **Adapt deliberately** and document the adaptation (P-18).
4. **Verify** the adaptation cannot be bypassed unintentionally.

**Rule:** an encounter that becomes soloable through an unintended bypass is a **defect**, logged and fixed
— never quietly accepted as emergent design.

`[A]` Blocked until V-1: the 3.3.5a tuning state of level-60 encounters must be measured before any
adaptation is designed.

---

## 7. Open questions affecting this document

`Q2` acquisition model (shapes stages 1–3 entirely) · `Q3` power source — existing spells cross-chassis vs
original effects — which determines whether stage 5 is usually skipped (curation project) or usually the
bulk of the work (effects project) · `Q7` nerf policy (governs stage 8).
Full list: requirements §14.
