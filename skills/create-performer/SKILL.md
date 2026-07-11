---
name: create-performer
description: Write a single job performer note into an Obsidian vault's `performers/` folder, using the canonical performer shape (frontmatter `tags: [performer]`, `main_job` linking to the job they execute, optional `also_performs` for secondary jobs, a functional prose definition, a Distinct From section naming adjacent roles they are not, and a Context of Execution section). Use when the user wants to add one performer to an existing vault — "add a performer for the weekly planner job", "define the executor of capture an idea" — and as the file-writing primitive that the `model-performers` skill delegates to once a confirmed performer set has been agreed. This skill owns the artifact shape; the parent `model-performers` skill owns the conversation and the multi-performer workflow. Performers are defined functionally: by the job they execute, not by who they are — no demographics, no personal names, no psychographics.
---

# Create Performer

Writes one performer note to `<vault>/performers/<PerformerName>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-performer "<Name>"`) or delegated to by `model-performers` once a confirmed performer has been chosen.

## When to use

- The user names a single performer to define: "add the executor for 'plan my week'", "define the collaborator role".
- A parent skill (`model-performers`) has confirmed a performer set and needs to write each file.
- The user is iterating in an existing vault and wants one more performer defined quickly.

Do **not** trigger from cold context where the user hasn't yet identified a vault or agreed on the performer set — that's `model-performers`'s job. This skill assumes those decisions are made.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, see [[model-performers]] step 1 to discover it.
2. **Performer name** — a functional role label, not a personal name. Title-case noun phrase describing the act: `The Planner`, `The Capturer`, `The Collaborator`. Used as the filename.
3. **Main job** — `[[wiki link]]` to the single job this performer is the executor of. The performer is *defined* by this job — it is not optional.
4. **Also performs** — zero or more `[[wiki links]]` to secondary jobs this functional role also executes. Omit if none.
5. **Functional definition** — 2–3 sentences describing the act this performer executes, from the functional angle. No demographics, no personal context, no personality. If you find yourself writing "someone who is X type of person," stop and reframe around the act.
6. **Distinct from** — 2–4 bullets naming adjacent roles that are explicitly NOT this performer, with a one-line explanation of each. Common adjacent roles: buyer, approver, reviewer, manager, delegator, audience, assistant. Only list the ones that are actually present in this product's ecosystem.
7. **Context of execution** — 1–2 sentences: the situation that triggers this performer role. Frame it as the triggering condition, not the person's lifestyle: "when the backlog is unbounded and needs bounding" not "on Sunday evenings."

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/performers/` exists, `<vault>/Templates/Performer.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `performer` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

### 2. Write the performer file

Write `<vault>/performers/<PerformerName>.md` with this shape:

```markdown
---
tags:
  - performer
main_job: "[[Job sentence]]"
also_performs:
  - "[[Another job sentence]]"
---

<2–3 sentences defining this performer functionally: the act they execute, starting from the functional objective and working outward. No demographics, no personal characteristics, no psychographics. The definition should remain true regardless of which specific individual fills this role.>

## Distinct from

- **<Adjacent role>** — <Why they are not the performer of this job — what their relationship to the job actually is.>
- **<Adjacent role>** — ...

## Context of execution

<1–2 sentences: the triggering condition that activates this functional role — the situation, not the person. Frame it as "when X happens" rather than "the type of person who does X.">
```

Rules for filling it in:

- **`tags`** is always `[performer]`. Never omit.
- **`main_job`** is a single `[[wiki link]]`. A performer with two equally primary jobs is a sign they should be two performers — check with the user.
- **`also_performs`** — omit the key entirely if empty, not `also_performs: []`.
- **Filename** — title-case functional role label: `The Planner.md`, `The Collaborator.md`. Never a personal name (`Alex.md`), never a demographic descriptor (`The Millennial.md`).
- **Functional definition prose** — the test: could you substitute a different individual and have the definition still apply? If yes, it's functional. If it depends on personal traits, reframe.
- **Distinct from** — only list roles that a reader might confuse with this performer. Don't list every possible role in the ecosystem.
- **Don't overwrite** — if the file already exists, stop and ask the user whether to replace, merge, or skip.

### 3. Report

Print the path written, the main job linked, any secondary jobs linked, and any unresolved `[[wikilinks]]` (jobs that don't yet exist — candidates for the next call to `model-jobs`).

The canonical `Templates/Performer.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
