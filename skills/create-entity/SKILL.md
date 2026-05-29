---
name: create-entity
description: Write a single entity note into an Obsidian vault's `entities/` folder, using the canonical entity shape (frontmatter `tags: [entity]` + `relations:` block, prose elaboration, fields table, optional relationship notes and invariants). Use when the user wants to add one entity to an existing vault — "add an entity called X", "create a Notification entity", "model just one more thing" — and as the file-writing primitive that the `entity-modeling` skill delegates to once a full entity set has been proposed and confirmed. This skill owns the artifact shape; the parent `entity-modeling` skill owns the conversation and the multi-entity workflow.
---

# Create Entity

Writes one entity note to `<vault>/entities/<EntityName>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-entity <Name>`) or delegated to by `entity-modeling` once a confirmed entity has been chosen.

## When to use

- The user names a single entity to add: "add an Invoice entity", "create User.md".
- A parent skill (`entity-modeling`) has confirmed an entity set and needs to write each file.
- The user is iterating in an existing vault and wants one more entity stood up quickly.

Do **not** trigger this skill from cold context where the user hasn't yet identified a vault or proposed the broader entity set — that's `entity-modeling`'s job. This skill assumes those decisions are made.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, see [[entity-modeling]] step 1 to discover it.
2. **Entity name** — PascalCase singular (`Invoice`, `LineItem`, not `invoices`).
3. **Prose elaboration** — 1–3 sentences explaining what the entity is, the role it plays, and why it exists. References to other entities with `[[wiki links]]` are encouraged.
4. **Relations** — grouped by kind (`belongs_to`, `has_one`, `has_many`, `has_many_through`), each a list of `"[[OtherEntity]]"` strings. Any kind can be omitted if empty.
5. **Fields** — list of `(name, type, notes)` triples. Include `id`, `created_at`, `updated_at` by default unless the caller says otherwise.
6. *(Optional)* **Relationship notes** — prose for ambiguous role names, optionality, cascade rules.
7. *(Optional)* **Invariants** — rules that must always hold.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

1. Confirm `<vault>/entities/` exists (`mkdir -p` if not).
2. Confirm `<vault>/Templates/Entity.md` exists. If not, create it with the template at the bottom of this skill so future hand-creation in Obsidian uses the same shape. Do **not** overwrite an existing template.
3. Confirm `<vault>/.obsidian/templates.json` points the core Templates plugin at `Templates/`. If the file is missing, write `{ "folder": "Templates" }`. If it exists with a different value, leave it.

### 2. Write the entity file

Write `<vault>/entities/<EntityName>.md` with this shape:

```markdown
---
tags:
  - entity
relations:
  belongs_to:
    - "[[OtherEntity]]"
  has_one:
    - "[[OtherEntity]]"
  has_many:
    - "[[OtherEntity]]"
  has_many_through:
    - "[[OtherEntity]]"   # via <JoinEntity>
---

<1–3 sentence prose elaboration: what this entity represents, the role it plays in the product, and why it exists.>

## Fields

| Field         | Type                 | Notes                                |
| ------------- | -------------------- | ------------------------------------ |
| id            | UUID                 | Primary key                          |
| ...           | ...                  | ...                                  |
| created_at    | datetime             |                                      |
| updated_at    | datetime             |                                      |

## Relationship notes

- <Optional. Use only when a relationship needs prose. Delete the section if every relation in the frontmatter speaks for itself.>

## Invariants

- <Optional. Rules that must always hold. Delete this section if there's nothing to say.>
```

Rules for filling it in:

- **`tags`** is always `[entity]`. Never omit — this is how the vault's tag pane groups all entities.
- **`relations`** — omit a kind entirely if it has no entries; don't write `belongs_to: []`. Omit the whole block if the entity has no relations.
- **Filename** — PascalCase singular, exact match to the entity name. `Task.md`, never `tasks.md`.
- **Don't overwrite** — if `<vault>/entities/<EntityName>.md` already exists, stop and ask the user whether to replace, merge, or skip.

### 3. Report

Print the path written, the relations created, and any unresolved `[[wikilinks]]` (entities referenced that don't exist yet — these are candidates for the next call).

## Reference template

The file written to `<vault>/Templates/Entity.md` if it's missing — identical shape to the body above but with `{{title}}` placeholders so Obsidian's core Templates plugin can insert it into a new note:

```markdown
---
tags:
  - entity
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
