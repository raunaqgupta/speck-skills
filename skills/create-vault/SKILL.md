---
name: create-vault
description: Resolves or creates the Obsidian vault a Speck-modeled product lives in — finds an existing vault (parent-resolved path, user-named location, the in-repo default, or a pre-existing standalone vault in Obsidian's registry) and, if none exists, bootstraps a new one with the shared scaffolding every entity, job, performer, workflow, or system note needs: the `entities/`, `jobs/`, `performers/`, `workflows/`, and `systems/` folders, all five canonical `Templates/*.md` files, `.obsidian/templates.json` pointing the core Templates plugin at `Templates/`, and `.obsidian/graph.json` color groups for each of the five tags. Idempotent: safe to call before every write, only fills in what's missing, never overwrites existing template content or touches unrelated `.obsidian` settings (zoom, physics, search filter, existing color groups). Use directly when the user wants to find or start a vault ("where's the vault for this?", "set up a vault for this", "create an empty vault at X") — and as the first thing every `create-entity`, `create-job`, `create-performer`, `create-workflow`, and `model-*` skill delegates to for vault resolution, instead of each looking to a sibling skill for that logic.
---

# Create Vault

Resolves an existing vault or bootstraps a new one at `<vault>`, fully scaffolded for Speck. Self-contained: can be invoked directly by a user (`/create-vault [path]`) or delegated to by any other skill in this plugin as its first step, before it does anything of its own — no skill should look to another skill (e.g. `model-entities`) for vault-resolution logic; they all come here, so this stays the one place it's written and no skill implicitly depends on another having run first.

## When to use

- Any skill in this plugin — `create-entity` / `create-job` / `create-performer` / `create-workflow`, any `model-*` skill, or `speck` — needs a vault path before it can do its own work, whether that means finding an existing one or creating a new one.
- The user asks directly: "where's the vault for this?", "set up a vault for this", "initialize a vault for X".

Do **not** use this to write an entity, job, performer, workflow, or system note — that's the job of the kind-specific `create-*` skill. This skill only resolves and ensures the vault itself is ready to receive one.

## Inputs

1. **Vault path** — absolute path to the vault root. Resolve it in this order:
   1. If delegated to by another skill that already resolved a vault path, use that as-is.
   2. If the user named a location directly, use it.
   3. Otherwise, check whether the current repo already has one: resolve the **default path** below and check whether `<default path>/.obsidian` exists. If it does, use it.
   4. Otherwise, check Obsidian's config for a pre-existing standalone vault that predates the in-repo convention:
      ```bash
      cat "$HOME/Library/Application Support/obsidian/obsidian.json"
      ```
      (On Linux: `~/.config/obsidian/obsidian.json`. On Windows: `%APPDATA%\obsidian\obsidian.json`.)

      The JSON lists vaults by path; entries with `"open": true` are the user's currently active vaults. If exactly one open vault clearly matches the product context (e.g. its path contains the product name), propose it. Otherwise list the candidates and ask.
   5. If no vault exists for this product anywhere, use the **default path** below. Propose it to the user and confirm before creating anything there — never create it silently.

### Default path

The vault lives *inside the code repo being modeled*, not in some separate standalone location. Find the repo root (`git rev-parse --show-toplevel` from the current working directory; if that fails, the current working directory itself), take its basename as `<repo-name>`, and default to `<repo-root>/speck-<repo-name>/`.

This is the single source of truth for that formula — no other skill re-derives it.

## Workflow

### 1. Ensure the vault directory and `.obsidian/` exist

```bash
mkdir -p "<vault>/.obsidian"
```

If `<vault>` already existed with content in it (an existing vault, or a non-empty non-vault directory), proceed anyway — this step and the ones below are additive and never touch unrelated files.

### 2. Ensure the five kind folders exist

```bash
mkdir -p "<vault>/entities" "<vault>/jobs" "<vault>/performers" "<vault>/workflows" "<vault>/systems"
```

### 3. Ensure all five templates exist

For each of `Entity`, `Job`, `Performer`, `Workflow`, `System`: if `<vault>/Templates/<Kind>.md` does not already exist, create `<vault>/Templates/` (if needed) and write it using the canonical body from the Reference section below. **Never overwrite an existing template** — if it's already there, leave it untouched even if its shape looks outdated; that's the user's customization to keep or change themselves.

Every template's frontmatter uses `tags: [template]`, never the kind-specific tag (`entity`, `job`, etc.). This is deliberate: real notes are tagged with their kind so the graph view and tag pane can group them, while template files are tagged `template` so `.obsidian/graph.json`'s `search: "-tag:#template"` filter (set up in step 5) hides them from the graph instead of appearing as phantom nodes.

### 4. Ensure `.obsidian/templates.json` points at `Templates/`

If `<vault>/.obsidian/templates.json` is missing, write:

```json
{ "folder": "Templates" }
```

If it already exists with a different value, leave it — the user may have a reason for a different template folder.

### 5. Ensure `.obsidian/graph.json` has the template filter and a color group for each kind

Each of the five kinds gets a fixed, stable color so the graph reads consistently across every vault this plugin sets up:

| Kind | `query` | `rgb` |
| --- | --- | --- |
| entity | `tag:#entity  ` | `14048348` |
| job | `tag:#job  ` | `14069084` |
| performer | `tag:#performer` | `11392604` |
| workflow | `tag:#workflow` | `6084188` |
| system | `tag:#system` | `6084269` |

(The five values rotate the same three magnitudes — `214`, `173`, `92` — through the R/G/B channels, so the five kinds land at evenly spaced points around the color wheel: red, orange, yellow-green, green, spring-green.)

- If `<vault>/.obsidian/graph.json` doesn't exist, create it with:
  ```json
  {
    "collapse-filter": false,
    "search": "-tag:#template",
    "colorGroups": [
      { "query": "tag:#entity  ", "color": { "a": 1, "rgb": 14048348 } },
      { "query": "tag:#job  ", "color": { "a": 1, "rgb": 14069084 } },
      { "query": "tag:#performer", "color": { "a": 1, "rgb": 11392604 } },
      { "query": "tag:#workflow", "color": { "a": 1, "rgb": 6084188 } },
      { "query": "tag:#system", "color": { "a": 1, "rgb": 6084269 } }
    ]
  }
  ```
- If it already exists, don't just check whether the file is present — Obsidian itself writes a bare-default `graph.json` (empty `search`, empty `colorGroups`) the moment it indexes a new vault folder, often before this step runs, and that bootstrap default is not the same thing as a user's deliberate customization. Treat `colorGroups` and `search` accordingly, each on its own terms:
  - **`colorGroups`** — create the array if the key is missing. Check for an entry whose `query` matches `tag:#<kind>` for each of the five kinds (ignore trailing whitespace differences when matching), and append an entry from the table above for any kind that's missing one. Never reorder or rewrite existing entries — those may be the user's own additions or recoloring.
  - **`search`** — if it's empty or the key is missing, set it to `-tag:#template`. Only leave it as-is if it already holds some other non-empty value; a non-empty value is a real signal of deliberate customization, an empty one is just Obsidian's own unconfigured default and isn't something to preserve.
  - **Leave every other field untouched** — `scale`, physics settings (`centerStrength`, `repelStrength`, `linkDistance`, ...), `collapse-filter`, `showOrphans`, and similar panel-state toggles. Unlike `search`, these are cosmetic display preferences rather than functional filters, so there's no bug in leaving Obsidian's own bootstrap values for them alone.

### 6. Report

Print what was created (folders, templates, config files) versus what already existed and was left alone. If this was a genuinely empty directory before step 1, say so plainly: "Bootstrapped a new vault at `<path>`."

## Reference: the five templates

Each block below is written verbatim to `<vault>/Templates/<Kind>.md` when that file is missing.

### `Templates/Entity.md`

```markdown
---
tags:
  - template
relations:
  belongs_to:
    - "[[ ]]"
  has_one:
    - "[[ ]]"
  has_many:
    - "[[ ]]"
  has_many_through:
    - "[[ ]]"   # via [[ ]]
---

A 1–3 sentence prose elaboration on what `{{title}}` is, the role it plays in the product, and why it exists. Reference other entities with `[[wiki links]]` where natural.

## Fields

| Field      | Type     | Notes       |
| ---------- | -------- | ----------- |
| id         | UUID     | Primary key |
|            |          |             |
| created_at | datetime |             |
| updated_at | datetime |             |

## Relationship notes

- Use this section only when a relationship needs prose. Delete the section if every relation in the frontmatter speaks for itself.

## Invariants

- Rules that must always hold. Delete this section if there's nothing to say.
```

### `Templates/Job.md`

```markdown
---
tags:
  - template
situation: "<When does this job arise? The trigger — time of day, life event, recurring context.>"
outcome: "<The success state from the user's POV. 'User feels X' or 'User has Y'.>"
touches:
  reads:
    - "[[ ]]"
  creates:
    - "[[ ]]"
  updates:
    - "[[ ]]"
forces:
  push:
    - "<Pain with the status quo that pushes them to switch>"
  pull:
    - "<Promise of the new solution that pulls them>"
  habit:
    - "<What they'd have to give up — familiarity, sunk cost>"
  anxiety:
    - "<What they fear might go wrong with the switch>"
---

A 2–4 sentence prose description of `{{title}}`: who's doing it, what they're trying to accomplish, and the headline of the switch. Reference entities inline where natural.

## Tasks

1. <First task> — touches [[Entity]]
2. <Second task>
3. ...

## Success criteria

- <Optional. Observable signal the job is done well.>
```

### `Templates/Performer.md`

```markdown
---
tags:
  - template
main_job: "[[ ]]"
also_performs:
  - "[[ ]]"
---

A 2–3 sentence functional definition of `{{title}}`: the act they execute, starting from the functional objective. No demographics or personal characteristics — the definition should hold regardless of which individual fills this role.

## Distinct from

- **<Adjacent role>** — <Why they are not the performer of this job.>
- **<Adjacent role>** — ...

## Context of execution

<The triggering condition that activates this functional role — the situation, not the person.>
```

### `Templates/Workflow.md`

```markdown
---
tags:
  - template
system: "[[ ]]"
job: "[[ ]]"
performer: "[[ ]]"
touches:
  reads:
    - "[[ ]]"
  creates:
    - "[[ ]]"
  updates:
    - "[[ ]]"
---

A 1–3 sentence prose description of `{{title}}`: what it accomplishes, where it starts, where it ends. Omit the `job` frontmatter key entirely if this workflow has no job behind it.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | | | *(not yet designed)* |
| 2 | | | *(not yet designed)* |

## Notes

- Optional. Branching, ordering constraints, or open design questions. Delete if there's nothing to say.
```

### `Templates/System.md`

```markdown
---
tags:
  - template
kind: product
consumes:
  - "[[ ]]"
consumed_by:
  - "[[ ]]"
---

A 2–3 sentence description of what `{{title}}` is responsible for: its boundary, and — if it has consumers other than the end user — the job it does for them.

## Not responsible for

- **<Adjacent concern>** — owned by [[OtherSystem]] instead.

## Provides

- [[Job sentence]] — for <consumer>
```
