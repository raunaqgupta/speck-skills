---
name: create-job
description: Write a single job-to-be-done note into an Obsidian vault's `jobs/` folder, using the canonical job shape (frontmatter `tags: [job]`, `situation`, `outcome`, `touches` block linking entities by verb, all four Christensen forces, prose description, and a numbered Tasks list). Use when the user wants to add one job to an existing vault — "add a job for onboarding", "create a 'recover from overwhelm' job" — and as the file-writing primitive that the `model-jobs` skill delegates to once a confirmed job set has been agreed. This skill owns the artifact shape; the parent `model-jobs` skill owns the conversation and the multi-job workflow.
---

# Create Job

Writes one job note to `<vault>/jobs/<Sentence cased filename>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-job "<job sentence>"`) or delegated to by `model-jobs` once a confirmed job has been chosen.

## When to use

- The user names a single job to add: "add a 'review my finances' job", "create one for onboarding a teammate".
- A parent skill (`model-jobs`) has confirmed a job set and needs to write each file.
- The user is iterating in an existing vault and wants one more job stood up quickly.

Do **not** trigger from cold context where the user hasn't yet identified a vault or framed the broader JTBD picture — that's `model-jobs`'s job. This skill assumes those decisions are made.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, see [[model-jobs]] step 1 to discover it.
2. **Job sentence** — a verb phrase from the user's voice, used as the filename. Sentence case, no PascalCase: `Plan my week`, `Capture an idea`, `Recover from overwhelm`.
3. **Situation** — the concrete trigger moment. "Sunday evening, looking at the week ahead" beats "during weekly planning."
4. **Outcome** — the success state from the user's POV, not the product's. "User feels confident about Monday" not "App shows weekly view."
5. **Entities touched** — grouped by verb: `reads`, `creates`, `updates`. Each a list of `"[[EntityName]]"` strings. Omit a verb-list entirely if empty.
6. **The four forces** — `push`, `pull`, `habit`, `anxiety`. Each a list of short phrases. All four are required; if one genuinely can't be named, write `"(none identified — revisit after user interviews)"` rather than fabricating.
7. **Prose description** — 2–4 sentences describing who's doing the job, what they're trying to accomplish, and the headline of the switch.
8. **Tasks** — numbered list of the user-visible steps the user walks through, each ideally mentioning the entity it touches.
9. *(Optional)* **Success criteria** — observable signals that the job is done well, when not obvious from `outcome`.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/jobs/` exists, `<vault>/Templates/Job.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `job` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

### 2. Write the job file

Write `<vault>/jobs/<Job sentence>.md` with this shape:

```markdown
---
tags:
  - job
situation: "<When does this job arise? The trigger.>"
outcome: "<The success state from the user's POV.>"
touches:
  reads:
    - "[[Entity]]"
  creates:
    - "[[Entity]]"
  updates:
    - "[[Entity]]"
forces:
  push:
    - "<Pain with the status quo>"
  pull:
    - "<Promise of the new solution>"
  habit:
    - "<What they'd have to give up>"
  anxiety:
    - "<What they fear might go wrong>"
---

<2–4 sentence prose description of the job: who's doing it, what they're trying to accomplish, the headline of the switch.>

## Tasks

1. <First task> — touches [[Entity]]
2. <Second task>
3. ...

## Success criteria

- <Optional. Observable signal the job is done well.>
```

Rules for filling it in:

- **`tags`** is always `[job]`. Never omit.
- **Filename** is sentence-cased verb phrase — `Plan my week.md`, never `PlanMyWeek.md`, never `Weekly planning.md`.
- **`touches`** — omit a verb-list (`reads`, `creates`, `updates`) entirely if empty.
- **`forces`** — all four kinds are required entries. Write `"(none identified — revisit after user interviews)"` instead of dropping a force, so the gap is visible.
- **Don't overwrite** — if the job file already exists, stop and ask whether to replace, merge, or skip.

### 3. Report

Print the path written, the entities touched (split by verb), and which forces, if any, were flagged as unidentified.

The canonical `Templates/Job.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
