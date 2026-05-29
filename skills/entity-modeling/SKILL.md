---
name: entity-modeling
description: Model the domain entities (data model / schema / objects) of a product the user is planning to build, by creating one Markdown note per entity inside an Obsidian vault's `entities/` folder. Use this skill whenever the user is brainstorming, scoping, or planning a new product, app, feature, or service and the conversation touches on what "things" the system will have — users, items, records, entities, models, tables, resources, nouns. Trigger even if the user doesn't say "Obsidian" or "data model" explicitly: phrases like "I'm thinking about building…", "let's plan out…", "what entities would this need?", "model this app", "design the schema for…", or any product-planning discussion where naming the core nouns would unblock progress. Skill creates a durable, browsable knowledge graph (each entity links to related entities via `[[wiki links]]`) that the user can iterate on in Obsidian while they design.
---

# Entity Modeling

This skill turns product-planning conversations into a navigable set of entity notes in an Obsidian vault. Each note describes one domain entity: its fields, its relationships to other entities, and any invariants. Because relationships use Obsidian's `[[wiki links]]`, the result becomes a clickable, graph-viewable schema the user can refine over time.

## Why this exists

Naming the nouns is the cheapest, highest-leverage step in product planning. Once "User", "Task", "Project" exist as concrete artifacts, downstream conversations (API design, DB schema, UI screens, permissions) get a lot easier. Doing this in Obsidian — rather than a one-off doc or a whiteboard photo — means the model is persistent, linkable from other notes, and visualizable in the graph view.

## When to use

Trigger this skill when the user is **planning** a product and discussing what it contains. Examples of triggering moments:

- "I want to build a habit tracker — what should I think about?"
- "Let's design the data model for this."
- "What entities would a marketplace app need?"
- "I'm scoping out a CRM, can you help me think through it?"
- The user has just described a product idea in 2–3 sentences and is about to start implementing.

Do **not** trigger when the user is asking you to write code, generate an ER diagram in another tool, or modify entities in an existing codebase. This skill is for early-stage modeling, not implementation.

## Workflow

### 1. Identify the vault

The notes need a home. In order of preference:

1. If the user named a vault, use it.
2. Otherwise, check Obsidian's config to find available vaults:
   ```bash
   cat "$HOME/Library/Application Support/obsidian/obsidian.json"
   ```
   (On Linux: `~/.config/obsidian/obsidian.json`. On Windows: `%APPDATA%\obsidian\obsidian.json`.)

   The JSON lists vaults by path; entries with `"open": true` are the user's currently active vaults. If exactly one open vault clearly matches the product context (e.g. its path contains the product name), propose it. Otherwise list the candidates and ask.
3. If no vault exists for this product, offer to create one. A vault is just a directory with an empty `.obsidian/` folder inside:
   ```bash
   mkdir -p "<path>/<VaultName>/.obsidian"
   ```

### 2. Propose the entity set before writing

Read the product description back to yourself and draft a list of 4–8 core entities. Lean toward the smallest set that captures the product's nouns — it's easier to add later than to prune.

Common patterns to consider (use only if they fit):
- **User / Account / Profile** — almost always present
- **The core noun** — the thing the product is fundamentally about (Task, Recipe, Listing, Patient, etc.)
- **Container** — Project, Workspace, Collection, Folder
- **Categorization** — Tag, Label, Category
- **Social / collaboration** — Comment, Reaction, Mention, Invite
- **Time-based** — Reminder, Event, Notification, Schedule
- **Money** — Account, Transaction, Invoice, Subscription

Present the proposed list to the user with a one-line rationale per entity, and confirm before writing files. Let them add, remove, or rename. This is the single most valuable interaction in the skill — a list the user shapes is a list they own.

### 3. Create one entity at a time via `/create-entity`

For each confirmed entity, delegate the file-writing to the [[create-entity]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [entity]` + `relations:` block, prose elaboration, fields table, optional sections), handles vault prerequisites (`entities/` folder, `Templates/Entity.md`, `.obsidian/templates.json`), and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per entity, passing:

- Vault path
- Entity name (PascalCase singular)
- Prose elaboration (1–3 sentences)
- Relations, grouped by kind (`belongs_to`, `has_one`, `has_many`, `has_many_through`) as lists of `"[[EntityName]]"` strings
- Fields list, with `id`, `created_at`, `updated_at` as defaults
- Optional relationship notes and invariants

Recommended order: write the entities that have no `belongs_to` first (root-of-graph entities like `User`), then their dependents. This keeps `[[wikilinks]]` resolved as files appear in the vault. Self-references (a `Task` that belongs to a parent `Task`) and forward references are still fine — Obsidian resolves them once both files exist.

If a user interrupts the flow ("wait, also add a Notification entity"), just call `/create-entity` once more — that's the whole point of the separation. No need to re-run the proposal step.

### 4. Show the user what was created

After writing, list the files and offer next steps:

- Add more entities they think are missing
- Drill into any entity to flesh out fields, edge cases, or invariants
- Create an Obsidian canvas linking the entities visually
- Move on to API endpoints or UI screens (which can also live in the vault, in sibling folders like `endpoints/` or `screens/`)

## Using `obsidian-cli` (optional)

If `obsidian-cli` is installed (`command -v obsidian-cli`), it can list and inspect vault contents, but writing files directly via the `Write` tool is simpler and more reliable. Use `obsidian-cli list <path> --vault <VaultName>` to verify the result after writing if useful.

## Style notes

- One entity per file. Resist the urge to bundle.
- PascalCase singular names (`User`, `LineItem`) — not plurals, not snake_case.
- Link liberally with `[[wiki links]]`. A link to an entity that doesn't exist yet is fine — it appears as an "unresolved" link in Obsidian and signals "we should model this next."
- Don't invent fields the user hasn't implied. A todo app doesn't need `oauth_provider` unless someone said "sign in with Google."
- Don't write implementation code, DDL, or ORM models. This skill stops at the conceptual layer.
