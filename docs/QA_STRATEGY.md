# QA Strategy

**Status:** Proposed — awaiting owner approval.
**Last updated:** 2026-09-06
**Principle:** "It compiles" is not evidence. Neither is "it should work."

---

## 1. The evidence rule

**No build, test, migration or in-game check is reported as passing unless it was actually run and its
output is shown.**

This binds the agent without exception. If a check was skipped, that is stated explicitly. If a check
failed, the failure and its output are reported — not summarised away, not deferred to "will fix". A phase
is not complete because the work is done; it is complete when the verification is shown.

---

## 2. Test layers and ownership

| Layer | Location | Owns | Runs |
|---|---|---|---|
| **Unit** | `src/test/` (GoogleTest) | Eligibility rules, tag matching, budget arithmetic, curve maths, config validation — anything pure | Every change |
| **Live e2e** | `e2e/suites/` (Go, AzerothGhost harness) | Grant/removal, persistence across logout, prestige, both death paths, combat effects, multi-character interaction | Player-visible change |
| **Scratch e2e** | `e2e/local/` (gitignored) | Debugging only. Never committed. Keepers are promoted to `e2e/suites/` | Ad hoc |
| **SQL validation** | `pending_db_*/` against a scratch DB | Applies cleanly, is idempotent, rollback proven | Every migration |
| **Manual in-game** | Owner | Feel, legibility, UX under a stock client | Player-facing change |
| **Exploit / abuse** | Live e2e + review | One negative test per row of the anti-exploit register | Any system that could cause one |
| **Performance** | Instrumented live run | Combat-path budget (T-7), telemetry growth (TM-3) | Any combat-path or high-volume change |
| **Balance regression** | Telemetry queries | M-1…M-11 against the §2 bands of `BALANCE_FRAMEWORK.md` | Any tuning change |

`[V]` The live-stack harness already exists: `e2e/README.md`, suites for combat, spells, quests, items,
instances, guild, social and protocol, importing `github.com/azerothcore/AzerothGhost` v1.0.8. Offline
`go test ./...` without `-tags=e2e` skips them.

**Consequence worth stating plainly:** because this harness exists, "verified in game" can mean
*automated and repeatable*, not merely "I logged in and it looked fine". That raises the achievable bar and
we should use it.

---

## 3. Mandatory gates by change type

| Change type | Unit | Live e2e | SQL validation | Manual | Exploit | Performance |
|---|---|---|---|---|---|---|
| Pure logic (rules, maths, parsing) | **required** | — | — | — | — | — |
| Player-visible behaviour | if logic exists | **required** | — | **required** | if applicable | — |
| New persistent table or column | — | **required** | **required** | — | — | if hot path |
| Acquisition / grant path | **required** | **required** | if schema | **required** | **required** (X-8) | — |
| Augment trigger system | **required** | **required** | — | **required** | **required** (X-3, X-4) | **required** (T-7) |
| Combat-path hook | **required** | **required** | — | **required** | — | **required** (T-7) |
| Tuning value change | — | — | — | — | — | balance regression |
| Telemetry addition | — | **required** | **required** | — | — | **required** (TM-3) |
| **Prestige reset path** | **required** | **required** | **required** | **required** | **required** (D7-a) | **required** |
| **Client AddOn change** (ADR-0013) | — | **required** | — | **required** | — | if per-frame work |
| **Augment trigger or combat hook** (D31-d) | **required** | **required** | — | — | **required** (X-3, X-4) | **required** — and a review check that it did **not** add a new hook registration |
| **First persistent table** (D15-b) | — | **required** | **required** | — | — | — · **blocked until a tested backup/restore exists** |

A change type not listed defaults to the strictest comparable row.

**`[O]` Backups gate the first persistent table (D15-b).** ADR-0015 puts the database in a Docker named
volume, `ac-database`, which `docker compose down -v` destroys. T-6 gives persistent progression no reset
valve and D7-a gives the prestige reset no undo, so there is no recovery path other than a backup. A
**tested restore** — not merely a configured backup — is a prerequisite to creating the first persistent
table. **ADR-0018 settles the policy** (encrypted logical dumps of `acore_auth` and `acore_characters`, with
the restore asserted automatically in the same job) and **ADR-0033 settles the cadence** — every 15 minutes,
aligned to `PlayerSaveInterval`. Per D18-c the gate still stands: the policy is designed but cannot yet be
exercised, and only a restore that has actually run satisfies this.

**`[O]` ADR-0033 adds two testable properties to the life-end write path.** **D33-c** requires the
`character_life` ending to be written *immediately*, not at the next `PlayerSaveInterval` tick — which is
assertable in e2e: kill a staked character, read the row before any periodic save could have run. **⚠ D33-b**
requires the same ending to reach an append-only record that a database restore does not roll back, and that
is only meaningfully tested by the restore job itself: restore a dump taken *before* a recorded death and
assert the death still stands. Until that assertion exists, "a death is never undone" is a claim, not a
verified property.

**`[O]` The AddOn is a new test layer this document does not yet cover.** ADR-0013 (D13-c) adds a Lua
codebase with its own release cycle, version-locked to the server. A stale AddOn misreporting resources is
worse than no AddOn at all, so version negotiation and refusal-to-run-when-mismatched need explicit tests.
This section needs expanding once the AddOn exists.

### 3.1 The prestige reset manifest (D7-a)

ADR-0007 makes prestige reset a live character vessel in place. It mutates a real row and its dependants,
with no undo, and a table missed is a permanent corruption or exploit. `[V]` `data/sql/base/db_characters/`
ships 108 table files, 35 named `character_*`, and 51 declaring a `guid int unsigned` column — not all of
which mean *character* guid.

**No prestige implementation may begin until a reset manifest exists** that classifies every relevant table
as **cleared**, **preserved**, or **archived to the life record**, derived from the schema in this checkout.
**It must also cover state that is not a table:** per D20-d, drafted proficiency **skills** and the client
**proficiency bitmask** both survive a naive reset and would hand the next life a silent permanent advantage
and cited `file.sql:line` — not from memory. The manifest is reviewed as a document in its own right, and
each classification needs a stated reason.

Two properties must be tested explicitly, not assumed:

- **Completeness** — after a reset, no run-scoped state survives on the vessel. Tested by populating every
  table in the *cleared* set on a live vessel, prestiging, and asserting each is empty.
- **Atomicity / resumability (D7-b)** — a reset interrupted part-way must leave the vessel in a state that is
  unambiguously detectable and recoverable. A half-reset vessel is both corruption and an exploit vector.

---

## 4. Anti-exploit testing

Every row of the anti-exploit register (`BALANCE_FRAMEWORK.md` §5) requires **at least one negative e2e
test** — a test that asserts the exploit *cannot* happen — before the system that could cause it ships.

Priority order, by architectural irreversibility rather than by likelihood:

1. **X-4 recursive damage/healing loops** and **X-3 infinite resource loops** — the trigger-depth guard and
   re-entrancy guard must exist in the augment system's *first commit*. Retrofitting these into a shipped
   trigger system is how servers get duplication bugs.
2. **X-8 reward duplication** — idempotency and transactionality tested at the grant path, not observed
   after the fact.
3. **X-9 reset abuse** and **X-10 trading/boosting** — tested at the life-end and persistence boundaries.
4. **X-1 dead starting kits** — an automated viability sweep over every legal starting kit, not spot checks.
5. **X-5, X-6, X-7** — uptime and count assertions.
6. **X-2, X-11** — detected primarily by telemetry rather than by test.

---

## 5. Test data and environment

- Accounts are created by the e2e harness (GM level 3). `[V]` Real player accounts are never reused —
  `e2e/README.md`.
- Scratch and debug tests live in `e2e/local/` (gitignored) and are **never** committed. Anything worth
  keeping is promoted into `e2e/suites/` with proper metadata.
- `[V]` A live stack requires authserver + worldserver + MySQL with `acore_auth`, `acore_characters` and
  `acore_world`. Standing this up is a Phase 1 prerequisite alongside azerothMCP.

---

## 6. Migration and rollback

Every schema change states, in its plan, before it is written:

1. What it creates or alters.
2. How it rolls back.
3. What happens to existing accounts and lives — this project has **no reset valve**, so a migration that
   cannot be reversed is a permanent decision (T-6).
4. Which lifecycle events it must survive: life end, prestige, character deletion, account deletion.

`[V]` Migrations land only in `data/sql/updates/pending_db_*/`. `base/`, `archive/` and merged `db_*/` are
immutable (`AGENTS.md`).

---

## 7. Self-review

Every phase ends with a hostile self-review before it is presented as complete, covering: regressions,
exploits, security, performance, balance, missing tests, schema mistakes, and unclear player experience.
Repository rules for this live in `.agents/docs/self-review-rules.md` and `.agents/docs/code-review.md`.

The self-review report states what was verified **and what was not**. An unverified area named honestly is
worth more than a claim of completeness.

---

## 8. Definition of done

A change is done when all of the following are true and shown:

- [ ] Mandatory gates for its change type (§3) have been run, with output.
- [ ] Anti-exploit negative tests exist for any register row it could touch.
- [ ] Telemetry exists for anything it will later be balanced on (Pillar 5).
- [ ] Migration and rollback are stated and the rollback has been exercised.
- [ ] Documentation and decision log are updated.
- [ ] Self-review completed, including what was *not* verified.
- [ ] Owner has reviewed the diff and confirmed in-game testing.
