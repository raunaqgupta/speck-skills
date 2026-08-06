---
name: model-workflows
description: Model the concrete interaction workflows of a product — the ordered steps, screens, and entity-state transitions that carry out a job, or that stand alone with no job behind them (routine login, logout, session refresh) — by creating one Markdown note per workflow inside an Obsidian vault's `workflows/` folder. Use this skill whenever the user is designing or documenting how an interaction actually happens: "walk through the login flow", "what are the steps to capture a task", "design the interaction for X", "map the screens for this", or any conversation about the concrete sequence a performer moves through, as opposed to why they're motivated to. Trigger even when no job exists yet — many workflows (authentication, onboarding chrome, settings) are pure interaction design with no job-to-be-done of their own. Pairs with model-jobs (a workflow optionally elaborates one job's `## Tasks` sketch into a full step sequence), model-performers (who executes the workflow), model-surfaces (which delivery boundary the workflow happens on — every workflow note cites one), and model-entities (the states each step transitions).
---

# Workflow Modeling

This skill turns interaction-design conversations into a navigable set of workflow notes in an Obsidian vault. Each note describes one concrete sequence of steps: what the performer does, what entity state changes, and (once designed) what screen it happens on. Workflows link to the [[jobs]] they elaborate, the [[performers]] who execute them, and the [[entities]] they mutate — closing the loop between *why* (job), *who* (performer), *what* (entity), and *how* (workflow).

## Why this exists

A job's `## Tasks` list is deliberately a sketch — a few prose lines proving the job is achievable, written before any screen exists. That's the right altitude for early JTBD modeling, but it can't answer "what does the user actually click" or "what state does the Task move through." Workflow is the layer that holds that detail, without forcing it into the job note (which would couple early demand-side modeling to late-stage UI decisions) or requiring it to wait for screens to be designed (a workflow's steps and transitions are real before any screen exists — screens are an optional annotation, not a precondition).

Workflow is also where interactions that **aren't** jobs get modeled. Login, logout, session refresh, and similar plumbing don't pass the JTBD test — nobody's making progress in their life by authenticating — so they have no home in `jobs/`. They still need to be designed and documented; workflow notes hold them without fabricating a situation/outcome/forces they don't have.

## When to use

Trigger this skill when the user is **designing or documenting a concrete interaction**. Examples:

- "Walk through the steps of capturing a task."
- "What's the login flow look like?"
- "Let's map out the screens for onboarding."
- "This job's Tasks list is too thin — flesh out the actual steps."
- The user is naming specific UI moments, state changes, or screen-to-screen transitions rather than motivations.

Do **not** trigger when the user wants to know *why* someone would use the product (use [[model-jobs]]) or *who* executes it (use [[model-performers]]) or *what nouns* the system has (use [[model-entities]]). Do **not** require a job to exist before modeling a workflow — check for one, but proceed without it if the interaction genuinely has none (see "Why this exists" above).

## Workflow

### 1. Identify the vault

Delegate to [[create-vault]] with no path (unless the user named one). It resolves an already-open or already-resolved vault, finds an existing in-repo or standalone vault, or proposes and creates a new one. Don't duplicate that logic here, and don't look to `model-entities` or any other `model-*` skill for it — `create-vault` is the one place it's written.

### 2. Check for existing jobs, performers, surfaces, and entities

Read the vault's `jobs/`, `performers/`, `surfaces/`, and `entities/` folders if present. For each candidate workflow:

- If it elaborates an existing job's `## Tasks` sketch, note that job as the anchor.
- If it's plumbing with no job behind it (auth, session handling, generic navigation chrome), proceed without one — don't force a job link to make the frontmatter look complete.
- Identify which performer executes it, if the vault has performer notes.
- Identify which surface it happens on — `create-workflow`'s `surface` field is always present, so this needs a real answer. If a job is reachable through more than one surface (a human via the web app, an AI session via an API), that's a signal for more than one workflow, one per surface, not one workflow trying to cover both.
- Identify which entities its steps will touch, so the `touches` block and transitions are grounded in real fields, not invented ones.

If `surfaces/` doesn't exist yet or doesn't cover the surface this workflow needs, offer to run [[model-surfaces]] first rather than guessing — don't invent a surface note reference that doesn't exist. If none of `jobs/`, `performers/`, or `entities/` exist yet either, workflows can still be modeled speculatively, but flag to the user that the vault is thin and offer to run [[model-entities]] first — a workflow with no entities to transition is hard to make concrete.

### 3. Propose the workflow set before writing

Draft one workflow per distinct interaction sequence worth documenting. A few rules:

- **One workflow per sequence, not per screen.** A workflow spans multiple steps/screens toward one accomplishment (capturing a task, logging in) — it isn't a single screen.
- **Don't default to one workflow per job.** Some jobs (`Triage my inbox`) may need none if the existing Tasks sketch is sufficient; others may need more than one if there are genuinely distinct paths to the same outcome — including one per surface, when a job is reachable more than one way.
- **Actively surface job-less workflows.** Scan for plumbing the vault hasn't captured anywhere — authentication, empty states, error recovery, settings — and propose them even though no job will anchor them.

For each candidate, present:
- Workflow name (verb phrase, matching job-naming convention)
- The job it elaborates, if any (or explicitly "no job — standalone interaction")
- The performer, if known
- The surface it happens on
- The entities its steps will likely touch

Present the list to the user and confirm before writing files. As with the other modeling skills, this proposal step is the most valuable interaction — it's where job-less workflows either get validated as genuinely job-less or get redirected back to [[model-jobs]] because a real situation/outcome was hiding underneath.

**Skip-confirmation mode.** If the user has explicitly said something like "don't ask for confirmation, just write it" or "stop confirming with me," skip the wait: still show the proposed list above, then move straight to writing instead of pausing for a reply. Unless they scope it narrower ("just for this one," "just for workflows"), treat it as a standing preference for the rest of the session — covering every `model-*` skill and `speck` call from here on, since re-stating it each time would defeat the point. It reverts the moment the user asks to confirm again. Never infer this from a fast or approving reply; it has to be requested explicitly.

### 4. Create one workflow at a time via `/create-workflow`

For each confirmed workflow, delegate the file-writing to the [[create-workflow]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [workflow]` + `surface` + optional `job`/`performer` + `touches`, prose description, Steps table with Action/Transition/Screen columns), delegates vault prerequisites (`workflows/` folder, `Templates/Workflow.md`, `.obsidian/templates.json`, graph color group) to [[create-vault]], links back from the anchor job's Tasks section, and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per workflow, passing:

- Vault path and surface
- Workflow name (sentence-cased verb phrase)
- Job (if one exists) and performer (if known)
- Entities touched, grouped by `reads` / `creates` / `updates`
- Prose description (1–3 sentences)
- Ordered steps: action, transition (or `—`), screen (or `*(not yet designed)*`)

Recommended order: write workflows that elaborate existing jobs first (they're the most grounded), then job-less/standalone workflows. If the user adds one mid-flow ("also map out password reset"), just call `/create-workflow` once more.

### 5. Show the user what was created

List the files. Offer next steps:

- Add workflows for any job whose Tasks sketch still feels too thin
- Flesh out `*(not yet designed)*` screens once real UI exists — this doesn't require rewriting the workflow, just filling in the Screen column
- Cross-check: every step's Transition should name a field that actually exists on the target entity's Fields table. Transitions that don't are candidates for revisiting [[model-entities]] first.
- Move on to designing the screens themselves, if the product is ready for that layer

## Style notes

- One workflow per file. Resist bundling "login" and "signup" into one note — different entry conditions, different steps.
- Sentence-cased verb phrases for filenames (`Log in`, `Capture a task`) — same convention as jobs, never PascalCase.
- A workflow does not need a job. Don't fabricate a situation/outcome/forces just to justify writing one — that's what [[model-jobs]] is for, and forcing plumbing into it produces hollow forces nobody believes.
- A workflow does not need its screens designed. Leave `*(not yet designed)*` rather than inventing screen names ahead of real design work.
- Don't invent entity transitions the entity's Fields table doesn't support. If a step needs a field that doesn't exist yet, that's a signal to revisit the entity, not to write a fictional transition.
- Don't write pixel-level UI spec, copy, or visual design here. This skill stops at the sequence-and-state layer.
