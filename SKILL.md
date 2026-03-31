---
name: aep
description: Experimental v1.0 of Robi AEP with richer schema (1.0-exp), better pack matching, confidence scoring, and pack evolution metadata. Still repo-native and file-based; use when you want more expressive, experimental AEP behavior on top of the stable v0.1 design.
icon: ./assets/aep-smithery-icon.svg
---

## Robi AEP Skill v1.0-exp

This is an **experimental extension** of the Robi AEP Skill.

Core idea remains the same:

- Convert successful collaborations into **Agent Experience Packs (AEPs)** stored in the active agent's directory.
- Use those packs to align future tasks.

v1.0-exp adds:

- richer **matching metadata** (`applies_to`, `strength`)
- **metrics** and **history** for how packs evolve
- optional **merge suggestions** between overlapping packs

Everything is still **repo-local** and **file-only**.

---

## Agent-Aware Install Target (Required)

Do **not** default to `.agent/` in v1.0-exp.

When creating or updating AEP files, first detect which agent environment is active for the repo, then install into that agent's directory.

Preferred targets:

- Claude: `.claude/aep/`
- Codex: `.codex/aep/`
- Gemini: `.gemini/aep/`
- OpenCode: `.opencode/aep/`
- Cursor: `.cursor/aep/`

If multiple agent directories exist, choose in this order:

1. directory explicitly referenced by the user
2. directory that already contains `aep/` files
3. directory matching the currently used agent
4. ask the user if still ambiguous

Do not create `.agent/` for new installs.

### Instruction file updates (Required after save/create)

After creating an AEP pack (or initializing AEP in a repo), update agent instruction files so future sessions know to use AEP by default.

Update/create these when present or relevant:

- `AGENTS.md`
- `CLAUDE.md`
- Codex instructions file (for the repo's Codex setup)
- Gemini instructions file (for the repo's Gemini setup)
- OpenCode agent instruction/config file
- Cursor project instruction/rules file

Minimum instruction to add (adapt path to active target):

- "Before starting tasks, load relevant AEP packs from `<agent-dir>/aep/`."
- "Apply task packs first, then project, then user packs."
- "After successful tasks, save new or updated AEP packs."

---

## Schema Version

All v1.0-exp packs use:

- `"version": "1.0-exp"`

See `aep-exp/references/schema.v1.md` for details.

---

## Additional Concepts

On top of the v0.1 fields, v1.0-exp introduces:

- `applies_to` – structured hints for where this pack makes sense:
  - `languages`, `frameworks`, `paths`, `domains`.
- `strength` – a single 0–1 score for how strong a match is for typical tasks.
- `metrics` – simple counters and timestamps about usage:
  - `times_applied`, `first_used_at`, `last_used_at`, `avg_turns_saved` (optional).
- `history` – short records of meaningful changes to the pack.
- `merge_suggestions` – optional hints that two packs should be merged or related.

These are **experimental** and not required for basic interoperability.

---

## Commands (Behavioral Layer)

The four high-level commands stay the same:

- `aep save`
- `aep apply`
- `aep promote`
- `aep inspect`

v1.0-exp refines how they populate and use the richer fields.

### 1. `aep save` (v1.0-exp)

When the user asks to save the current successful workflow:

1. **Extract signals** as in v0.1:
   - `intent`, `constraints`, `preferences`, `workflow`, `failure_traps`, `success_checks`.
2. **Infer `applies_to`** from:
   - languages and frameworks in use (e.g. `"typescript"`, `"nextjs"`, `"python"`, `"django"`).
   - relevant paths (e.g. `"app/landing/*"`, `"src/api/*"`).
   - domain hints (e.g. `"frontend"`, `"backend"`, `"infra"`).
3. **Initialize experimental fields**:
   - `strength`: start with a default (e.g. `0.7` for clearly successful tasks).
   - `metrics`:
     - `times_applied`: `0` on first creation.
     - `first_used_at`: equal to `created_at`.
     - `last_used_at`: equal to `created_at`.
   - `history`:
     - one entry describing the initial creation (reason, timestamp).
   - `merge_suggestions`: leave empty by default.
4. **Resolve install target**:
   - Detect active agent directory (`.claude`, `.codex`, `.gemini`, `.opencode`, `.cursor`).
   - Set AEP root to `<agent-dir>/aep/`.
5. **Write pack**:
   - Save as `<agent-dir>/aep/tasks/<id>.aep.json` with `"version": "1.0-exp"`.
   - Update `<agent-dir>/aep/index.json`:
     - include `version`, `scope`, `path`, `tags`, and `strength` if known.
6. **Update instruction files**:
   - Update `AGENTS.md`, `CLAUDE.md`, and other agent instruction files in this repo to state that AEP should be loaded by default from `<agent-dir>/aep/`.
7. **Optional evidence**:
   - Same as v0.1: `<agent-dir>/aep/runs/<timestamp-id>/` folder with:
     - `transcript.summary.md`, `signals.json`, `outcome.json`, `generated-pack.json`.

### 2. `aep apply` (v1.0-exp)

When applying packs before a task:

1. **Resolve install target**:
   - detect active agent directory and use `<agent-dir>/aep/` as source.
2. **Load packs** with `version` `"0.1"` or `"1.0-exp"` (backwards compatible).
3. **Compute a match score** per pack using:
   - keyword/tag overlap (as in v0.1),
   - `applies_to` matches:
     - languages / frameworks,
     - file paths / domains.
4. **Combine scores**:
   - final score = f(match_score, pack.`strength`).
5. **Rank**:
   - by scope (task > project > user),
   - then by final score,
   - then by recency (`metrics.last_used_at` or `index.updated_at`).
6. **Update metrics** for selected packs:
   - increment `metrics.times_applied`.
   - update `metrics.last_used_at`.
7. **Activate packs and explain**:
   - merge signals as in v0.1 (task overrides project overrides user).
   - report:
     - which packs are active,
     - their scores and `strength`,
     - key constraints and checks.

### 3. `aep promote` (v1.0-exp)

Promotion works as in v0.1 but can leverage:

- `metrics` and `history` to identify strong candidates:
  - e.g. packs with high `times_applied` and good user feedback.
- optional `merge_suggestions`:
  - when two task packs are similar, suggest merging before promotion.

When promoting:

- add history entries to both source and target packs.
- optionally record a `merge_suggestions` note if packs should be merged later.
- write promoted packs under the active `<agent-dir>/aep/` target, not `.agent/`.

### 4. `aep inspect` (v1.0-exp)

In addition to the v0.1 inspection:

- show:
  - `applies_to`
  - `strength`
  - `metrics` (how often and how recently a pack was used)
  - latest `history` entries
- highlight:
  - if any packs have `merge_suggestions` involving active packs.

---

## Behavior Rules (v1.0-exp)

- Maintain **backwards compatibility**:
  - be able to read v0.1 packs and index entries.
  - when updating, you may choose to **migrate** a pack to `"version": "1.0-exp"` (with user approval).
- Use **agent-aware paths**:
  - prefer `.claude/aep`, `.codex/aep`, `.gemini/aep`, `.opencode/aep`, `.cursor/aep`.
  - do not create `.agent/` for new installs.
- Keep instruction files aligned:
  - after creating/updating AEP, ensure `AGENTS.md`, `CLAUDE.md`, and other active agent instruction files state that AEP should be loaded first.
- Keep experimental fields **best-effort**:
  - it is fine if `metrics` or `history` are approximate.
  - do not block core behavior if some fields are missing.
- Remain **transparent and inspectable**:
  - everything stays JSON/Markdown.
  - users should be able to edit packs by hand if needed.

For concrete field definitions and templates, see:

- `aep-exp/references/schema.v1.md`
- `aep-exp/assets/templates/*.json`

