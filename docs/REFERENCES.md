# External references

**Status:** Active.
**Last updated:** 2026-09-04
**Purpose:** Where authoritative AzerothCore information lives, and how much weight each source carries.

The working agreement's rule 5 forbids inventing APIs, database fields, hooks, commands or core behaviour,
and requires verification against "the local source or official AzerothCore documentation". This document
says *where that documentation is* and *how to cite it*.

---

## 1. The AzerothCore wiki — primary external source

**<https://www.azerothcore.org/wiki/home>** — maintained by the AzerothCore project itself, in the
repository **<https://github.com/azerothcore/wiki>** (MIT, `master`, actively maintained). Every wiki page is
a markdown file under `docs/` in that repository, so the site and the source are the same content.

This is the **first place to look** for anything about how AzerothCore works that is not answerable from our
own checkout: module structure, hooks, the SQL update system, the DB schema, the spell system, build options,
GM commands, testing conventions and contribution process.

## 2. Local clone

The whole wiki is cloned locally so it can be read and searched offline and in full:

```
/home/chris/Projects/azerothcore-wiki
```

- **445 English pages**, ~346,000 words, plus a Spanish translation set under `docs/es/` and `docs/archive/`.
- Cloned `--depth 1` from `master`. Refresh with `git -C /home/chris/Projects/azerothcore-wiki pull`.
- Check what you have: `git -C /home/chris/Projects/azerothcore-wiki log -1 --format='%h %ad' --date=short`.

**It lives outside this repository on purpose.** Per ADR-0005 everything committed to the fork is public and
permanent; vendoring another project's MIT-licensed documentation into it would add noise and a licensing
question for no benefit. The clone is a local tool. This file — the pointer — is what is version-controlled.

Search it directly, e.g.:

```
grep -rl "SmartAI"  /home/chris/Projects/azerothcore-wiki/docs --include='*.md'
sed -n '1,60p'      /home/chris/Projects/azerothcore-wiki/docs/the-modular-structure.md
```

## 3. How much weight the wiki carries

The wiki is genuinely good. `[V]` Spot-checked on 2026-09-04: `docs/characters.md` documents all **80**
columns of the `characters` table and matches `data/sql/base/db_characters/characters.sql` in this checkout
exactly — no missing, extra or renamed columns.

It is nonetheless *documentation*, and it is not versioned against the commit we build from. So:

| Claim about… | Authoritative source | Cited as |
|---|---|---|
| Code behaviour, signatures, hooks, constants, config validation | **This checkout** | `file.ext:line` |
| Schema as we will actually migrate it | **This checkout** (`data/sql/`) | `file.sql:line` |
| Process and conventions — SQL versioning, PR and test process, module layout | **The wiki** | page name + URL |
| Orientation, intent, "what is this table/system for" | **The wiki** | page name + URL |
| Anything the wiki and the checkout disagree on | **This checkout wins** | cite both, note the divergence |

This does **not** relax rule 5 or T-2. A wiki page is a legitimate citation for process and intent; a claim
about what the code *does* still gets read out of the checkout and cited to a line.

## 4. What is in the wiki

| Group | Roughly | Notes |
|---|---|---|
| Per-table DB documentation | ~330 pages | One page per `world` / `characters` / `auth` table, column by column |
| Guides and concepts | ~90 pages | Indexed at [documentation-index](https://www.azerothcore.org/wiki/documentation-index) |
| Install and setup | ~12 pages | Requirements through client setup |
| Contribution and testing | ~8 pages | PR process, DB PRs, testing, triaging |
| Wiki meta / archive | remainder | Standards, callouts, superseded pages |

The published `documentation-index` lists only about 90 pages, so **the index is not the full contents** —
the per-table schema pages are reached from `database-world`, `database-characters` and `database-auth`, or
just found in the local clone.

## 5. Pages that bear on work already on our books

| Our item | Wiki pages |
|---|---|
| **ADR-0007 / D7-a reset manifest** | `database-characters`, `characters`, `character_spell`, and the other `character_*` table pages |
| `CORE_GAME_LOOP.md` §5.2 XP curve | `player_xp_for_level` |
| **Q2** acquisition model | `playercreateinfo` (+ `_spell_custom`, `_action`, `_item`, `_skills`), `character_spell`, `trainer_spell` |
| **Q3** where power comes from | `spell_system`, `spell-effects-reference`, `spell-aura-reference`, `spell_dbc`, `spell_script_names`, `spell_custom_attr` |
| **Q4** module hosting | `the-modular-structure`, `create-a-module`, `installing-a-module`, `hooks-script`, `hooks-cmake`, `hooks-bash` |
| Phase 1 build and tooling | `cmake-options`, `directory-structure`, `how-to-work-with-conf-files`, `logging-configuration` |
| `QA_STRATEGY.md` | `unit-testing`, `live-e2e`, `how-to-test-a-pr`, `how-to-test-db-only-changes` |
| SQL work (T-5, `pending_db_*`) | `sql-versioning`, `sql-directory`, `database-squash`, `how-to-create-a-db-pr` |
| Scripting conventions | `core-scripts`, `create-a-script`, `introduction-to-smartai` |
| Process and etiquette | `standard-operating-procedure`, `best-practices`, `contribute` |
| Our own working method | `agentic-engineering` — already linked from `../AGENTS.md` |

## 6. Other official sources

| Source | Use for |
|---|---|
| <https://www.azerothcore.org/doxygen> | Generated C++ API reference. Useful for orientation; the checkout is still authoritative |
| <https://github.com/azerothcore/azerothcore-wotlk/discussions/categories/guides-tips> | Community guides and tips |
| <https://wowdev.wiki/Category:DBC_WotLK> | DBC file format documentation (third-party, not AzerothCore) |
| <https://github.com/azerothcore/azerothMCP> | Read-only MCP server over the databases — see ADR-0003, adopted immediately before Phase 1 |
| `../doc/`, `../AGENTS.md`, `../.agents/docs/` | Upstream documentation and contributor rules shipped in this repository |
