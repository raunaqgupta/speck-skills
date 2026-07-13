---
name: create-workflow
description: Write a single workflow note into an Obsidian vault's `workflows/` folder, using the canonical workflow shape (frontmatter `tags: [workflow]`, optional `job` and `performer` links, a `touches` block, prose description, and a Steps table of action → entity-state transition → screen). Use when the user wants to add one workflow to an existing vault — "add a workflow for the login flow", "walk through the steps of capturing a task" — and as the file-writing primitive that the `model-workflows` skill delegates to once a confirmed workflow set has been agreed. This skill owns the artifact shape; the parent `model-workflows` skill owns the conversation and the multi-workflow proposal. A workflow is the concrete interaction sequence that carries out a job (or stands alone with no job behind it) — screens and states, not motivation.
---

# Create Workflow

Writes one workflow note to `<vault>/workflows/<Workflow name>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-workflow "<workflow name>"`) or delegated to by `model-workflows` once a confirmed workflow has been chosen.

## When to use

- The user names a single workflow to add: "add a workflow for logging in", "walk through the steps of setting a reminder".
- A parent skill (`model-workflows`) has confirmed a workflow set and needs to write each file.
- The user is iterating in an existing vault and wants one more workflow stood up quickly.

Do **not** trigger from cold context where the user hasn't yet identified a vault or framed the broader interaction picture — that's `model-workflows`' job. This skill assumes those decisions are made.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, see [[model-workflows]] step 1 to discover it.
2. **Workflow name** — a verb phrase from the user's voice, used as the filename. Sentence case, no PascalCase: `Capture a task`, `Log in`, `Set a reminder` — same convention as job filenames.
3. **System** — `[[wiki link]]` to the system this workflow belongs to, matching the `system` field already on entity, job, and performer notes in this vault.
4. **Job** *(optional)* — a single `[[wiki link]]` to the job this workflow carries out. Omit entirely if this workflow has no job behind it (routine login, logout, session refresh — interactions that don't pass the JTBD test on their own).
5. **Performer** *(optional)* — a single `[[wiki link]]` to the performer who executes this workflow. Omit if not yet known or not applicable.
6. **Entities touched** — grouped by verb: `reads`, `creates`, `updates`. Each a list of `"[[EntityName]]"` strings. Omit a verb-list entirely if empty.
7. **Prose description** — 1–3 sentences: what this workflow accomplishes, where it starts, where it ends.
8. **Steps** — an ordered list, each with:
   - **Action** — what the performer does, in plain language.
   - **Transition** *(optional)* — the entity-state change this step causes, as `[[Entity]]: <from> → <to>`. Use `—` if the step touches no entity state (e.g. a pure UI affordance like dismissing a modal).
   - **Screen** *(optional)* — `[[wiki link]]` to the screen this step happens on. Leave as `*(not yet designed)*` if no screen exists yet — a workflow does not require its screens to be designed first, and does not require them to ever exist as separate notes.
9. *(Optional)* **Notes** — branching, ordering constraints, or open design questions.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/workflows/` exists, `<vault>/Templates/Workflow.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `workflow` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

### 2. Write the workflow file

Write `<vault>/workflows/<Workflow name>.md` with this shape:

```markdown
---
tags:
  - workflow
system: "[[System]]"
job: "[[Job sentence]]"
performer: "[[Performer]]"
touches:
  reads:
    - "[[Entity]]"
  creates:
    - "[[Entity]]"
  updates:
    - "[[Entity]]"
---

<1–3 sentence prose description: what this workflow accomplishes, where it starts, where it ends.>

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | <what the performer does> | [[Entity]]: none → created | *(not yet designed)* |
| 2 | ... | — | *(not yet designed)* |

## Notes

- <Optional. Branching, ordering constraints, or open design questions. Delete if there's nothing to say.>
```

Rules for filling it in:

- **`tags`** is always `[workflow]`. Never omit.
- **Filename** is a sentence-cased verb phrase — `Log in.md`, never `LogIn.md`, never `Login flow.md`.
- **`system`** — always present, matching the convention on entity/job/performer notes.
- **`job`** — omit the key entirely (not a blank string) if this workflow has no job behind it. Don't force a job link to make the frontmatter look complete.
- **`performer`** — omit the key entirely if not yet known.
- **`touches`** — omit a verb-list (`reads`, `creates`, `updates`) entirely if empty.
- **Transition column** — write `[[Entity]]: <from> → <to>` when the step changes an entity's state (a field value, not necessarily a full row create/delete). Write `—` for steps that touch no entity state. Don't invent a transition to fill the cell.
- **Screen column** — write `*(not yet designed)*` rather than skipping the column or fabricating a screen name. A workflow is complete without its screens designed.
- **Don't overwrite** — if the workflow file already exists, stop and ask whether to replace, merge, or skip.

### 3. Link back from the job (if one exists)

If `job` is set, open that job's note and check its `## Tasks` section. If it doesn't already reference this workflow, append a line: `See [[Workflow name]] for the detailed step-by-step.` Don't duplicate the step list into the job note — the job's Tasks section stays the lightweight sketch; the workflow is the elaboration. If the job already links to a different workflow, ask before adding a second.

### 4. Report

Print the path written, the job and performer linked (or noted as absent), the entities touched, and any unresolved `[[wikilinks]]` (screens or entities referenced that don't exist yet).

The canonical `Templates/Workflow.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
