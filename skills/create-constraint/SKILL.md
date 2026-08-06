---
name: create-constraint
description: Write a single constraint note into an Obsidian vault's `constraints/` folder, using the canonical constraint shape (frontmatter `tags: [constraint]` + `applies_to:` links to the functional notes it binds, a 1–3 sentence boundary statement, a Rationale section, and an optional Implications section). Use when the user wants to add one implementation commitment to an existing vault — "add a constraint that auth must support SSO", "record that this has to run on-prem", "pin down the deployment model as a constraint" — and as the file-writing primitive that the `model-constraints` skill delegates to once a confirmed constraint set has been agreed. This skill owns the artifact shape; the parent `model-constraints` skill owns the conversation and the multi-constraint proposal. A constraint states what must hold, as a requirement — never a design — keeping implementation vocabulary (protocols, deployment modes, auth mechanisms) out of the entity/job/performer/workflow/system notes it binds.
---

# Create Constraint

Writes one constraint note to `<vault>/constraints/<Constraint statement>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-constraint "<statement>"`) or delegated to by `model-constraints` once a confirmed constraint has been chosen.

## When to use

- The user names a single implementation commitment to pin down: "add a constraint that the web app has to support external identity providers", "record that this must run both hosted and on-prem".
- A parent skill (`model-constraints`) has confirmed a constraint set and needs to write each file.
- The user is iterating in an existing vault and wants one more constraint captured quickly — typically reactive, when a specific implementation decision is being made, not part of a bulk brainstorming pass.

Do **not** trigger from cold context where the user hasn't yet identified a vault or agreed on the constraint set — that's `model-constraints`'s job. This skill assumes those decisions are made.

Do **not** use this for functional boundaries (what a system is and isn't responsible for) — that's [[create-vault]]'s System template. A constraint is specifically an *implementation* commitment: a deployment mode, a protocol choice, an auth mechanism — something that would read as out-of-place vocabulary inside an entity, job, performer, workflow, or system note.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, delegate to [[create-vault]] to resolve or create it.
2. **Constraint statement** — a short declarative sentence stating the requirement, sentence-case (capitalize only the first word and proper nouns/acronyms): `AI access is via MCP`, `Web-based, hosted or on-prem`. Used as the filename. Phrase it as a requirement ("X must Y" / "X is Z"), never as a design description.
3. **Applies to** — one or more `[[wiki links]]` to the functional notes (entities, jobs, performers, workflows, or systems) this constraint binds. Required — a constraint that applies to nothing isn't a constraint; if the caller can't name what it binds, it isn't ready to be written yet.
4. **Boundary statement** — 1–3 sentences stating what must hold, as a requirement rather than a design decision. The notes it applies to stay implementation-free; this is where that vocabulary lives instead.
5. **Rationale** — 1 or more sentences: why this constraint exists, the need or decision behind it.
6. *(Optional)* **Implications** — bullets of what the constraint rules in or out for the notes it applies to. Omit the section if there's nothing beyond the boundary statement worth spelling out.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/constraints/` exists, `<vault>/Templates/Constraint.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `constraint` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

### 2. Write the constraint file

Write `<vault>/constraints/<Constraint statement>.md` with this shape:

```markdown
---
tags:
  - constraint
applies_to:
  - "[[Bound note]]"
  - "[[Another bound note]]"
---

<1–3 sentences stating the boundary this constraint places on any implementation: what must hold, as a requirement rather than a design. The functional notes it applies to stay implementation-free; the implementation commitment lives here.>

## Rationale

<Why this constraint exists — the need or decision behind it.>

## Implications

- <What the constraint rules in or out for the notes it applies to.>
```

Rules for filling it in:

- **`tags`** is always `[constraint]`. Never omit.
- **`applies_to`** — one or more `[[wiki links]]`. Never empty; never `applies_to: []`.
- **Filename** — the constraint statement itself, sentence-case: `AI access is via MCP.md`, `Web auth supports external identity providers.md`. Never a generic label like `Auth Constraint.md` — the filename should be readable as the requirement on its own.
- **Boundary statement** — a requirement, not a design. "Auth must be able to delegate to a team's own identity provider" is a constraint; "we'll use NextAuth with an OIDC provider" is an implementation detail that belongs in `code/`, not the model. If you find yourself naming a specific product or library, check whether the model note should stay one level more abstract.
- **Implications** — omit the whole section (not just leave it empty) if the boundary statement and rationale already say everything worth saying.
- **Don't overwrite** — if the file already exists, stop and ask the user whether to replace, merge, or skip.

### 3. Report

Print the path written, the notes it applies to, and any unresolved `[[wikilinks]]` (notes that don't yet exist — candidates for the relevant `create-*`/`model-*` skill).

The canonical `Templates/Constraint.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
