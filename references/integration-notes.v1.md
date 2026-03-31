## Robi AEP v1.0-exp Integration Notes

This document explains how to integrate the **experimental v1.0-exp** Robi AEP schema into agents, building on the simpler v0.1 design.

v1.0-exp adds:

- `applies_to` metadata (languages, frameworks, paths, domains)
- `strength` scoring
- basic usage `metrics`
- `history` and `merge_suggestions`

Everything is still **repo-local** and **file-based**.

---

## Coexisting with v0.1

Recommended pattern:

- **Read** both v0.1 and v1.0-exp packs from `.agent/aep/`.
- **Prefer** v1.0-exp packs when both exist for the same domain.
- **Write** new experimental packs with `"version": "1.0-exp"` while keeping v0.1 packs stable for fallback.

Index (`.agent/aep/index.json`) may list mixed versions; see `schema.v1.md` for fields.

---

## Apply Flow (Scoring and Ranking)

When implementing `aep apply` with v1.0-exp:

1. **Load candidates**
   - From `.agent/aep/index.json` (if present).
   - For each entry, load the referenced pack.
   - Include both `"0.1"` and `"1.0-exp"` packs.

2. **Compute a base match score** for each pack:
   - Use task description + repo context.
   - Match against:
     - `match.keywords`
     - `match.patterns`
     - `match.tags`
   - Normalize score into \[0, 1].

3. **Adjust with `applies_to`** (v1.0-exp only):
   - If languages/frameworks/paths/domains clearly align, **boost** the score.
   - If they clearly mismatch, **dampen** the score.
   - Keep behavior soft; avoid hard disqualification based on `applies_to` alone.

4. **Combine with `strength`**:
   - If `strength` is present, compute something like:
     - `combined_score = 0.6 * match_score + 0.4 * strength`
   - If missing, treat strength as neutral (e.g. 0.5).

5. **Scope-aware ranking**:
   - Sort primarily by:
     1. scope: `task` > `project` > `user`
     2. `combined_score`
     3. recency: `metrics.last_used_at` or `index.updated_at`

6. **Select active packs**:
   - Typically 1–3 packs.
   - Ensure at least one has strong alignment; otherwise prefer fewer packs.

7. **Update metrics** (if allowed):
   - Increment `metrics.times_applied`.
   - Update `metrics.last_used_at`.

8. **Explain to the user**:
   - Which packs were selected.
   - Their combined scores and key `applies_to` hints.

---

## Save Flow (Evolving Packs)

When the user asks to save a new pattern or update an existing one:

1. **Determine whether to create or update**:
   - If there is a clear existing pack with strong overlap, consider:
     - updating that pack, or
     - creating a new pack and later suggesting a merge.
   - Otherwise, create a new pack.

2. **Populate experimental fields**:
   - `applies_to` based on:
     - languages/frameworks you touched.
     - paths involved in the changes.
     - domain (frontend/backend/infra/etc.).
   - `strength`:
     - default moderately high for clear success (e.g. `0.7`–`0.8`).
   - `metrics`:
     - initialize `times_applied` to `0` on creation (or `1` if you treat this run as first application).
     - set `first_used_at` and `last_used_at` to the same timestamp.
   - `history`:
     - add a `created` or `updated` event with a short reason.

3. **Write pack and index**:
   - Save with `"version": "1.0-exp"`.
   - Ensure `.agent/aep/index.json` has a corresponding entry, including:
     - `version`, `strength`, `updated_at`.

4. **Runs evidence**:
   - Optionally store the run under `.agent/aep/runs/<timestamp-id>/` as in v0.1.

---

## Promotion and Merge Suggestions

v1.0-exp encourages you to track **pack evolution** and potential merges.

### Promotion

When promoting:

- From `task` → `project`:
  - Extract constraints and preferences that have worked across multiple tasks.
  - Create/update `project.aep.json` with `"version": "1.0-exp"`.
  - Add a `history` entry in both source and target packs.

- From `project` → `user`:
  - Focus on collaboration style and high-level defaults.

### Merge Suggestions

When two packs overlap heavily:

- Rather than merging immediately, add `merge_suggestions` entries like:

```json
{
  "target_id": "landing-page-migration",
  "reason": "Intent, constraints, and match metadata are highly similar."
}
```

This lets humans or higher-level tools decide when/how to merge.

---

## Inspect Flow (Surfacing Experimental Data)

When implementing `aep inspect` for v1.0-exp, show:

- active packs with:
  - `id`, `scope`, `version`, `title`
  - `applies_to` summary
  - `strength` and key metrics:
    - `times_applied`, `last_used_at`, `avg_turns_saved` if present
- constraints, preferences, and success checks currently in force
- recent `history` entries
- any `merge_suggestions` involving active packs

This makes the experimental scoring and evolution features understandable to users.

---

## Platform Notes

- **Cursor/Codex-style agents**:
  - v1.0-exp can be adopted purely as a richer JSON format.
  - The agent logic lives entirely in prompts + file operations.

- **Claude Code / OpenCode**:
  - v1.0-exp can be used identically to v0.1 from the file perspective.
  - Future MCP servers can optionally expose structured operations that understand `strength`, `metrics`, and `history`.

In all cases, keep v1.0-exp **opt-in** and experimental:  
do not break existing v0.1-based workflows. 

