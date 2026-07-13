---
name: speck
description: Model an entire product end-to-end in one command — entities, jobs, performers, and workflows — by sequencing the model-entities, model-jobs, model-performers, and model-workflows skills in dependency order against a single vault, then running a second pass that applies each skill's own cross-checks against the now-complete model to catch gaps the forward-only first pass couldn't see (orphan entities no job touches, jobs no performer executes, workflow transitions naming entity fields that don't exist, thin Tasks sketches a workflow has since elaborated). Use when the user wants the full picture in one go rather than one layer at a time — "model this whole product", "let's fully map this app out", "set up the complete model for X: data, jobs, performers, and workflows", "I want everything mapped, not just the entities". If the user only wants a single layer ("just the entities", "what jobs would this need?"), trigger that layer's own model-* skill directly instead — this skill is specifically for requests that span multiple or all layers at once.
---

# Speck

Runs the full modeling pipeline against one vault in two passes. The first pass writes entities, then jobs, then performers, then workflows — each layer delegated to its own skill, in the dependency order that lets later layers reference earlier ones (jobs touch entities, performers execute jobs, workflows carry out jobs via performers and touch entities). The second pass goes back over all four layers using the cross-checks each `model-*` skill already defines, now that the full model exists to check against — catching the gaps a strictly-forward pass structurally can't see, like a workflow needing an entity field nothing proposed yet, or a job whose Tasks sketch stayed thin until a workflow made it concrete. This skill sequences and reconciles; it doesn't own any layer's artifact shape or proposal logic itself.

## When to use

- The user describes a product and wants it modeled comprehensively, not one layer at a time: "model this whole product", "map out the full picture for a habit tracker", "I want the data model, jobs, performers, and workflows all set up."
- The user has already modeled some layers and asks to fill in the rest: still trigger this skill, but see step 0 — it skips layers that are already substantially populated rather than re-proposing them.
- The user has already run `speck` (or modeled all four layers manually) and wants it reconciled or double-checked: trigger this skill again — step 0 will find all four layers already populated and skip straight to step 5's second pass.

Do **not** trigger this for single-layer requests ("what entities would this need?", "let's define the performers") — those go to the specific `model-*` skill, which is more precisely scoped for that conversation.

## Workflow

### 0. Identify the vault once, and detect what's already there

Delegate to [[create-vault]] with no path (unless the user named one) to resolve a single vault path for this whole run. Every subsequent skill call below reuses that same path — none of them should re-run vault discovery.

If the resolved vault already has notes in one or more of `entities/`, `jobs/`, `performers/`, `workflows/`, tell the user which layers already have content and confirm whether to skip those and only run the empty ones, or re-propose additions on top of what's there. Don't silently skip or silently re-propose — ask.

If all four already have notes, there's nothing left to propose — confirm with the user, then skip straight to step 5, the second pass. This is how re-running `speck` on an already-modeled vault turns into a reconciliation pass instead of redoing work.

### 1. Model entities

Delegate to [[model-entities]], passing the resolved vault path (skip its own vault-discovery step 1 entirely). Propose the entity set, confirm with the user, write each one via `create-entity`.

Do not proceed to step 2 until this layer is confirmed and written — jobs reference these entities.

### 2. Model jobs

Delegate to [[model-jobs]], passing the resolved vault path. Propose the job set — referencing the entities just confirmed — confirm with the user, write each one via `create-job`.

### 3. Model performers

Delegate to [[model-performers]], passing the resolved vault path. Propose performers for the confirmed jobs, confirm, write each one via `create-performer`.

### 4. Model workflows

Delegate to [[model-workflows]], passing the resolved vault path. Propose workflows for the confirmed jobs, performers, and entities, confirm, write each one via `create-workflow`.

### 5. Second pass: refine using what the full model now reveals

Don't skip this step or treat it as optional busywork. Each `model-*` skill's own "Show the user what was created" step already defines a cross-check against its upstream layers, but on a single forward pass each check only ever runs once, immediately after its own layer, as a suggestion the user may never act on — it never gets revisited with what the *later* layers went on to reveal. This step is where that gets reconciled, using checks that already exist, not new logic invented here.

Now that all four layers exist, walk them again in the same order, but scoped only to what each check flags — this is a targeted patch pass, not a second full proposal round:

1. **Entities** — per [[model-jobs]]'s cross-check: any entity no job touches? Per [[model-workflows]]'s cross-check: any entity field a step's Transition needed that the Fields table doesn't have?
2. **Jobs** — per [[model-performers]]'s cross-check: any job with no performer executing it? Per [[model-workflows]]'s "Show what was created" step: any job whose `## Tasks` sketch stayed thin despite a workflow now elaborating it?
3. **Performers** — any performer whose `main_job` turned out to need splitting once a workflow made its steps concrete (two genuinely different executors hiding behind one job)?
4. **Workflows** — any job or performer that clearly needs a workflow and still has none?

Present the findings as one consolidated list grouped by layer, and confirm with the user before applying any of it — a flagged gap isn't automatically a bug. A support job genuinely having no performer, or plumbing genuinely having no job, are valid outcomes the checks will still flag; don't "fix" what the user confirms is intentional.

For each confirmed refinement, delegate back to the specific skill that owns that layer — add a field via [[create-entity]] (editing the existing note, not rewriting it), add a job via [[create-job]], and so on. Run this pass once; if the user wants to iterate further after seeing the result, they can just run `speck` again rather than looping automatically here.

### 6. Report

Summarize what was created across all four layers — counts per kind and the vault path — plus anything skipped in step 0 and anything refined in step 5.

## Rules

- **Never collapse the confirmations.** Each layer, and the second pass, still gets its own propose-then-confirm round. This skill sequences and reconciles, it doesn't replace either with one unreviewed dump of every note across every kind.
- **Respect a mid-pipeline stop.** If the user says "just entities and jobs for now" — during step 0 or after any layer completes — stop there instead of forcing all four and the second pass.
- **A flagged gap is a question, not an automatic edit.** The second pass surfaces what the cross-checks find; it doesn't silently delete orphan entities or silently invent a performer to fill a hole.
- **Don't rewrite any layer's artifact shape, proposal heuristics, or cross-check logic inline here.** Those belong to `model-entities` / `model-jobs` / `model-performers` / `model-workflows` respectively — this skill only re-invokes them at a point in time (after the full model exists) when their checks have more to work with.
