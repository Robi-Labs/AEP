## Robi AEP Schema v1.0-exp

This is an **experimental** schema for Robi AEP packs.

It extends the v0.1 schema with:

- richer **matching metadata** (`applies_to`)
- a `strength` score
- light-weight **metrics** and **history**
- optional **merge_suggestions**

All v1.0-exp packs use:

- `"version": "1.0-exp"`

Backwards compatibility: agents **should** still be able to read v0.1 packs.

---

## Top-Level Shape (Pack)

```json
{
  "version": "1.0-exp",
  "id": "html-to-nextjs-migration",
  "scope": "task",
  "title": "HTML to Next.js migration",
  "status": "approved",
  "source": {
    "type": "successful_collaboration",
    "created_at": "2026-03-31T10:00:00Z",
    "confidence": 0.91
  },
  "match": {
    "keywords": ["html", "nextjs", "migration", "template"],
    "patterns": [
      "convert static html to nextjs",
      "preserve design while migrating frontend"
    ],
    "tags": ["frontend", "migration", "nextjs", "html"]
  },
  "applies_to": {
    "languages": ["typescript"],
    "frameworks": ["nextjs"],
    "paths": ["app/**", "pages/**"],
    "domains": ["frontend"]
  },
  "strength": 0.9,
  "intent": [],
  "constraints": [],
  "preferences": [],
  "workflow": [],
  "failure_traps": [],
  "success_checks": [],
  "metrics": {
    "times_applied": 3,
    "first_used_at": "2026-04-01T10:00:00Z",
    "last_used_at": "2026-04-10T15:30:00Z",
    "avg_turns_saved": 6.0
  },
  "history": [
    {
      "at": "2026-03-31T10:00:00Z",
      "event": "created",
      "reason": "Initial successful HTML to Next.js migration"
    },
    {
      "at": "2026-04-05T09:15:00Z",
      "event": "updated",
      "reason": "Promoted constraint to avoid redesign without explicit request"
    }
  ],
  "merge_suggestions": [
    {
      "target_id": "landing-page-migration",
      "reason": "Very similar intent and constraints; consider merging packs."
    }
  ],
  "artifacts": {
    "examples": [],
    "notes": "Use this pack when preserving fidelity matters more than architectural cleanup."
  }
}
```

---

## Field Reference (New / Extended)

All fields from v0.1 remain valid:

- `version`, `id`, `scope`, `title`, `status`
- `source`, `match`
- `intent`, `constraints`, `preferences`, `workflow`
- `failure_traps`, `success_checks`
- `artifacts`

v1.0-exp adds or extends:

### `applies_to` (object, optional but recommended)

Hints for when this pack is most relevant.

Fields:

- `languages` (string[])
  - language names like `"typescript"`, `"python"`, `"go"`.
- `frameworks` (string[])
  - e.g. `"nextjs"`, `"django"`, `"rails"`.
- `paths` (string[])
  - glob-like hints such as:
    - `"app/**"`, `"src/api/**"`, `"infra/**"`.
- `domains` (string[])
  - domain labels like `"frontend"`, `"backend"`, `"infra"`, `"data"`.

Agents should treat these as **soft filters** when deciding if a pack applies to a task.

### `strength` (number, optional)

Single scalar in \[0, 1] representing how strong a pattern this pack encodes.

Interpretation:

- `0.3` – weak or tentative pattern.
- `0.7` – solid pattern that has worked multiple times.
- `0.9+` – very strong, highly trusted pattern.

Agents may combine this with a task-specific match score to rank packs:

- `combined_score = f(match_score, strength)`

### `metrics` (object, optional)

Simple usage metrics.

Fields:

- `times_applied` (integer)
  - how many times this pack has been actively applied to tasks.
- `first_used_at` (string, ISO 8601)
  - timestamp of first use.
- `last_used_at` (string, ISO 8601)
  - timestamp of most recent use.
- `avg_turns_saved` (number, optional)
  - rough estimate of how many back-and-forth messages this pack saves on average.

Agents **may** update these after each `aep apply`.

### `history` (array, optional)

Records meaningful changes to the pack.

Each entry:

```json
{
  "at": "2026-04-05T09:15:00Z",
  "event": "updated",
  "reason": "Promoted constraint to avoid redesign without explicit request"
}
```

Fields:

- `at` (string, ISO 8601) – when the event happened.
- `event` (string) – `"created"`, `"updated"`, `"promoted"`, `"merged"`, etc.
- `reason` (string) – short human-readable explanation.

### `merge_suggestions` (array, optional)

Hints that two or more packs should be merged or related.

Each suggestion:

```json
{
  "target_id": "landing-page-migration",
  "reason": "Very similar intent and constraints; consider merging packs."
}
```

Fields:

- `target_id` (string) – id of the other pack.
- `reason` (string) – short explanation.

Agents should **not** auto-merge based on these; they are cues for human review or higher-level tooling.

---

## Index Schema v1.0-exp (`.agent/aep/index.json`)

The index can now hold mixed versions, but each entry can carry more metadata.

```json
{
  "version": "1.0-exp",
  "packs": [
    {
      "id": "task-html-to-nextjs",
      "scope": "task",
      "version": "1.0-exp",
      "path": "tasks/html-to-nextjs.aep.json",
      "tags": ["frontend", "migration", "nextjs", "html"],
      "strength": 0.9,
      "updated_at": "2026-03-31T10:00:00Z"
    }
  ]
}
```

Fields:

- `version` – index schema version, `"1.0-exp"` for this experimental format.
- `packs` – array of entries:
  - `id` – pack id.
  - `scope` – `"task" | "project | user"`.
  - `version` – `"0.1"` or `"1.0-exp"`.
  - `path` – relative path from `.agent/aep/`.
  - `tags` – same meaning as before.
  - `strength` – optional, mirrors pack `strength`.
  - `updated_at` – ISO timestamp.

Agents should gracefully handle entries that lack some of these fields (especially for migrated or older packs).

---

## Runs Schema v1.0-exp

Same file set as v0.1, with the option to include experimental fields inside:

- `transcript.summary.md`
- `signals.json`
- `outcome.json`
- `generated-pack.json`

You may add:

- simple metadata about:
  - how many turns were saved
  - whether the user explicitly requested future reuse

There is no strict v1.0-exp schema for runs; they are auxiliary evidence.

---

## Compatibility Guidelines

- **Reading**:
  - Agents should support both `"version": "0.1"` and `"version": "1.0-exp"`.
- **Writing**:
  - When creating new experimental packs, use `"1.0-exp"`.
  - When updating old packs:
    - either keep them on v0.1 and only adjust v0.1 fields, or
    - explicitly migrate them to v1.0-exp (with a `history` entry).
- **Graceful degradation**:
  - If an agent does not understand `applies_to`, `metrics`, or `history`, it can ignore them and rely on the v0.1 subset.

