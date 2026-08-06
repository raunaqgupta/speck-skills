---
name: model-constraints
description: Model the implementation commitments of a product — deployment modes, protocol choices, auth mechanisms, and similar boundaries a design must honor — by creating one Markdown note per constraint inside an Obsidian vault's `constraints/` folder, each linked via `applies_to` to the functional notes (entities, jobs, performers, workflows, surfaces) it binds. Use this skill when a specific implementation decision is being pinned down and needs to be captured so it can't silently drift: "this has to run on-prem too", "auth needs to support the customer's own SSO", "record that AI access only happens through MCP". Also trigger during a review pass to surface implementation commitments that came up in conversation but were never captured. Unlike entities/jobs/performers/workflows, constraints are typically proposed reactively, one or a few at a time, rather than brainstormed in bulk up front — most products only accumulate a handful. Pairs with every other layer: a constraint's whole purpose is to bind onto notes that must otherwise stay implementation-free.
---

# Constraint Modeling

This skill turns implementation decisions into a navigable set of constraint notes in an Obsidian vault. Each note states one requirement a design must honor — a deployment mode, a protocol choice, an auth mechanism — and links via `applies_to` to the functional notes it binds. Constraints are what let entity, job, performer, workflow, and surface notes stay implementation-free: instead of a surface note quietly picking up a stray mention of "hosted or on-prem" or "auth delegates to an IDP," that commitment gets its own note, and the functional note just links to it.

## Why this exists

Implementation decisions have a way of leaking into functional notes — a surface note mentions "runs on-prem," a workflow step says "via OAuth," and suddenly notes meant to describe *what* the product is are also describing *how* it's built, with no single place that decision lives or that other notes can point back to. Constraints exist to hold that vocabulary explicitly, separated out, each one naming exactly which functional notes it binds via `applies_to` — so a functional note stays legible to someone who cares only about behavior, while the implementation commitment is still captured, findable, and traceable to its rationale.

This is also why constraints behave differently from the other five layers. Entities, jobs, performers, surfaces, and workflows are usually modeled in a batch early on, because a product's nouns, motivations, roles, boundaries, and interactions mostly need to be sketched together to make sense of each other. Constraints don't work that way — they show up one at a time, at the moment a real implementation decision gets made ("we need to support the customer's own identity provider," decided mid-conversation), and forcing a bulk constraint-brainstorm before any implementation decisions exist would mean inventing constraints nothing has actually constrained yet.

## When to use

Trigger this skill when an implementation decision is being made or has just been made, and needs to be captured. Examples:

- "This has to work both hosted and self-hosted."
- "Auth needs to delegate to the customer's own identity provider, not just our login."
- "Record that AI clients only ever talk to this through MCP."
- A conversation about `code/` surfaces an implementation commitment that isn't yet a constraint note — capture it before moving on, rather than letting it live only in the conversation.
- A review pass (see `speck`'s cross-check) finds an implementation commitment mentioned somewhere that was never written down as a constraint.

Do **not** trigger for functional boundaries (what a surface is or isn't responsible for) — that's the Surface template via [[create-vault]], not a constraint. Do **not** trigger to brainstorm constraints speculatively with no real decision behind them — unlike the other five layers, an empty `constraints/` folder is a completely normal state for a product with no implementation commitments pinned down yet, not a gap to fill preemptively.

## Workflow

### 1. Identify the vault

Delegate to [[create-vault]] with no path (unless the user named one). It resolves an already-open or already-resolved vault, finds an existing in-repo or standalone vault, or proposes and creates a new one. Don't duplicate that logic here, and don't look to any other `model-*` skill for it — `create-vault` is the one place it's written.

### 2. Identify what the constraint binds

A constraint with nothing to bind isn't ready to be written. Check whether the notes it would apply to already exist:

- If the entity/job/performer/workflow/surface notes it binds already exist, proceed.
- If they don't exist yet, either offer to model them first via the relevant skill, or write the constraint with an unresolved `[[wikilink]]` if the user wants to capture the commitment now and fill in the binding later — flag it as unresolved either way, don't silently guess.

### 3. Propose the constraint(s) before writing

Unlike the other five layers, this is usually a proposal of one or two constraints, not a full-vault batch:

- **State it as a requirement, not a design.** "Auth must delegate to a team's own identity provider" is a constraint. "We'll use NextAuth with a generic OIDC provider" is an implementation detail for `code/`, not the model — if the proposed statement names a specific product, library, or vendor, push it back up one level of abstraction before proposing it.
- **Name every note it applies to.** A constraint binds one or more functional notes via `applies_to`; if you can't name at least one, it isn't ready.
- **Check for an existing constraint covering the same ground** before proposing a new one — read `constraints/` first. Two constraints that both bind the same note and say related things are a sign one should be extended instead of duplicated.

For each candidate, present:
- The constraint statement (filename)
- What it applies to
- A one-line rationale

Present the list (even if it's a single constraint) and confirm before writing files.

**Skip-confirmation mode.** If the user has explicitly said something like "don't ask for confirmation, just write it" or "stop confirming with me," skip the wait: still show the proposed constraint(s) above, then move straight to writing instead of pausing for a reply. Unless they scope it narrower, treat it as a standing preference for the rest of the session — covering every `model-*` skill and `speck` call from here on. Never infer this from a fast or approving reply; it has to be requested explicitly.

### 4. Create one constraint at a time via `/create-constraint`

For each confirmed constraint, delegate the file-writing to the [[create-constraint]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [constraint]` + `applies_to`, boundary statement, Rationale, optional Implications), delegates vault prerequisites (`constraints/` folder, `Templates/Constraint.md`, `.obsidian/templates.json`, graph color group) to [[create-vault]], and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per constraint, passing:

- Vault path
- Constraint statement (sentence-case requirement, used as the filename)
- Applies to (one or more `[[wiki links]]`)
- Boundary statement (1–3 sentences)
- Rationale
- Implications, if any

### 5. Show the user what was created

List the files. Offer next steps:

- Check whether the notes it applies to need a one-line pointer back (most don't need an explicit callout — the graph's backlinks already surface it — but a note with several constraints stacked on it might warrant a short mention in its own prose).
- Cross-check: does any other functional note mention implementation vocabulary (deployment, protocol, auth mechanism) that should have been pulled out into a constraint instead? That's `speck`'s cross-check territory when running the full pipeline, but worth a quick look here too.

## Style notes

- **One constraint per file, stated as a requirement.** Not "how we'll build it" — what must hold regardless of how.
- **Filename is the statement.** `AI access is via MCP.md`, not `MCP Constraint.md` — a reader should be able to understand the commitment from the filename alone.
- **`applies_to` is never empty.** A constraint that doesn't bind anything isn't a constraint yet.
- **Don't front-load constraints.** It's normal, even expected, for `constraints/` to stay empty or thin until real implementation decisions start getting made. Resist proposing constraints just because the other five layers are done.
- **Keep implementation vocabulary here, not in functional notes.** If you catch a job, entity, performer, workflow, or surface note using words like "database," "hash," "websocket," or "heartbeat," that's a sign the commitment belongs in a constraint instead.
