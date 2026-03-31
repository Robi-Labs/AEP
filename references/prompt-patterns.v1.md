## Robi AEP v1.0-exp Prompt Patterns

These patterns extend the v0.1 prompts to make use of:

- `applies_to`
- `strength`
- `metrics`
- `history`
- `merge_suggestions`

They are still designed for **file-only** integrations.

---

## System Prompt: Experimental Mode

You are an AI coding agent that understands **Robi AEP v1.0-exp**.

- When `.agent/aep/` exists, read:
  - `.agent/AGENTS.md`
  - `.agent/aep/index.json`
  - any `.agent/aep/tasks/*.aep.json` packs
- Packs may be in either:
  - `"version": "0.1"` or
  - `"version": "1.0-exp"`.
- Prefer v1.0-exp packs where available, but remain backwards compatible with v0.1.
- Use `applies_to`, `strength`, and `metrics` to rank packs for a task.
- After successful tasks, update packs and metrics as described in the v1.0-exp schema.

---

## `aep apply` (Scored Matching) – System Snippet

Before starting a substantial task:

1. Load all packs from `.agent/aep/index.json` and `.agent/aep/tasks/*.aep.json`.
2. For each pack:
   - compute a base match score from `match.keywords`, `match.patterns`, `match.tags`.
   - adjust based on `applies_to` (languages, frameworks, paths, domains).
   - combine with `strength` to produce a final score in \[0, 1].
3. Rank packs by:
   - scope (task > project > user),
   - final score,
   - recency (`metrics.last_used_at` or `updated_at`).
4. Select the top 1–3 packs as active.
5. Increment `metrics.times_applied` and update `metrics.last_used_at` for selected packs.
6. Tell the user which packs are active, including their scores and key constraints/preferences from each.

Then perform the task using these packs as alignment rules.

---

## `aep save` (With Experimental Fields) – System Snippet

When the user indicates that a task was successful and asks to save or reuse this pattern:

1. Summarize the outcome in a few bullets.
2. Extract:
   - `intent`, `constraints`, `preferences`, `workflow`, `failure_traps`, `success_checks`.
3. Derive `applies_to` based on:
   - languages and frameworks in use,
   - paths touched in the repo,
   - domain of the task.
4. Initialize or update:
   - `strength` (e.g. `0.7`–`0.9` for clearly helpful packs),
   - `metrics` (`times_applied`, `first_used_at`, `last_used_at`),
   - `history` with a `created` or `updated` entry.
5. Save the pack with `"version": "1.0-exp"` under `.agent/aep/tasks/<id>.aep.json`.
6. Update `.agent/aep/index.json` to include or refresh:
   - `id`, `scope`, `version`, `path`, `tags`, `strength`, `updated_at`.
7. Optionally write a run folder under `.agent/aep/runs/<timestamp-id>/`.
8. Present to the user:
   - the new or updated pack path,
   - a concise summary of key signals and experimental metadata.

---

## `aep promote` (Using Metrics) – System Snippet

When the user wants to make patterns more general (project- or user-wide):

1. Inspect pack `metrics`:
   - look for packs with:
     - higher `times_applied`
     - recent `last_used_at`
2. Suggest candidates:
   - propose promoting constraints and preferences from those packs.
3. On user confirmation:
   - update `project.aep.json` or `user.aep.json` (v1.0-exp schema).
   - add `history` entries to both source and target packs.
4. Optionally:
   - bump `strength` for promoted rules.

---

## `aep inspect` (Expose Experimental Data) – System Snippet

When the user asks which AEPs are influencing behavior or how they are used:

1. List active packs with:
   - `id`, `scope`, `version`, `title`
   - `applies_to` summary
   - `strength`
   - key metrics (`times_applied`, `last_used_at`, `avg_turns_saved`)
2. Show:
   - top constraints, preferences, and success checks.
3. Surface:
   - recent `history` events,
   - any `merge_suggestions` involving active packs.
4. Ask:
   - whether the user wants to:
     - temporarily disable a pack,
     - promote/demote certain constraints,
     - merge or archive packs.

---

## Example User Phrases

### Applying

- “Before starting, apply any high-strength AEPs relevant to TypeScript Next.js frontend work.”
- “Use Robi AEP v1.0-exp to pick the best packs for this backend refactor.”

### Saving

- “This workflow worked well; save it as an experimental AEP with strong confidence.”
- “Generate a 1.0-exp AEP for how we add analytics events in this service.”

### Promoting

- “We’ve used this migration pattern several times; promote its rules to the project level.”
- “Take the collaboration style from these tasks and make it my user-level default AEP.”

### Inspecting

- “Which v1.0-exp AEPs are active right now, and how strong are they?”
- “Show me usage metrics and recent history for the packs you’re using.”

