---
name: create-entity
description: Write a single entity note into an Obsidian vault's `entities/` folder, using the canonical entity shape (frontmatter `tags: [entity]` + `relations:` block, prose elaboration, fields table, optional relationship notes and invariants). Use when the user wants to add one entity to an existing vault — "add an entity called X", "create a Notification entity", "model just one more thing" — and as the file-writing primitive that the `model-entities` skill delegates to once a full entity set has been proposed and confirmed. This skill owns the artifact shape; the parent `model-entities` skill owns the conversation and the multi-entity workflow.
---

# Create Entity

Writes one entity note to `<vault>/entities/<EntityName>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-entity <Name>`) or delegated to by `model-entities` once a confirmed entity has been chosen.

## When to use

- The user names a single entity to add: "add an Invoice entity", "create User.md".
- A parent skill (`model-entities`) has confirmed an entity set and needs to write each file.
- The user is iterating in an existing vault and wants one more entity stood up quickly.

Do **not** trigger this skill from cold context where the user hasn't yet identified a vault or proposed the broader entity set — that's `model-entities`'s job. This skill assumes those decisions are made.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, delegate to [[create-vault]] to resolve or create it.
2. **Entity name** — PascalCase singular (`Invoice`, `LineItem`, not `invoices`).
3. **Prose elaboration** — 1–3 sentences explaining what the entity is, the role it plays, and why it exists. References to other entities with `[[wiki links]]` are encouraged.
4. **Relations** — grouped by kind (`belongs_to`, `has_one`, `has_many`, `has_many_through`), each a list of `"[[OtherEntity]]"` strings. Any kind can be omitted if empty.
5. **Fields** — list of `(name, type, notes)` triples. Include `id`, `created_at`, `updated_at` by default unless the caller says otherwise. Types are conceptual (`string`, `number`, `date`, `boolean`, "one of: a, b, c"), never database-implementation vocabulary — see the Rules below.
6. *(Optional)* **Relationship notes** — prose for ambiguous role names, optionality, cascade rules.
7. *(Optional)* **Invariants** — rules that must always hold.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/entities/` exists, `<vault>/Templates/Entity.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `entity` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

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
| id            | identifier           | unique identifier                    |
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
- **Fields table `Type` column is conceptual, never database-implementation vocabulary.** No `UUID` — write `identifier` (the conceptual fact is "uniquely identifies the record," already stated by "unique identifier" in Notes; the physical format is an implementation decision this skill doesn't make). No `FK -> [[Other]]` rows — that relationship is already fully expressed by the `relations:` block; a Fields table row for it just duplicates a frontmatter entry as a pretend foreign-key column. If a relation's optionality or nuance needs saying, say it in prose in `## Relationship notes`, not as a Fields row. No SQL `enum(...)` syntax — write "one of: a, b, c" instead; the list of valid states is real domain knowledge, the SQL syntax isn't. No `JSON` as a type — write "structured data" or similar; how it's serialized is an implementation choice.
- **Filename** — PascalCase singular, exact match to the entity name. `Task.md`, never `tasks.md`.
- **Don't overwrite** — if `<vault>/entities/<EntityName>.md` already exists, stop and ask the user whether to replace, merge, or skip.

### 3. Report

Print the path written, the relations created, and any unresolved `[[wikilinks]]` (entities referenced that don't exist yet — these are candidates for the next call).

The canonical `Templates/Entity.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
