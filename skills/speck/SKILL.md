---
name: speck
description: Model an entire product end-to-end in one command — entities, jobs, performers, workflows, and constraints — by sequencing the model-entities, model-jobs, model-performers, model-workflows, and model-constraints skills in dependency order against a single vault, then running a second pass that applies each skill's own cross-checks against the now-complete model to catch gaps the forward-only first pass couldn't see (orphan entities no job touches, jobs no performer executes, workflow transitions naming entity fields that don't exist, thin Tasks sketches a workflow has since elaborated, implementation vocabulary stranded in a functional note instead of pulled into a constraint). Use when the user wants the full picture in one go rather than one layer at a time — "model this whole product", "let's fully map this app out", "set up the complete model for X: data, jobs, performers, and workflows", "I want everything mapped, not just the entities". If the user only wants a single layer ("just the entities", "what jobs would this need?"), trigger that layer's own model-* skill directly instead — this skill is specifically for requests that span multiple or all layers at once.
---

# Speck

Runs the full modeling pipeline against one vault in two passes. The first pass writes entities, then jobs, then performers, then workflows, then checks for constraints — each layer delegated to its own skill, in the dependency order that lets later layers reference earlier ones (jobs touch entities, performers execute jobs, workflows carry out jobs via performers and touch entities, constraints bind onto any of the above). The second pass goes back over all five layers using the cross-checks each `model-*` skill already defines, now that the full model exists to check against — catching the gaps a strictly-forward pass structurally can't see, like a workflow needing an entity field nothing proposed yet, a job whose Tasks sketch stayed thin until a workflow made it concrete, or implementation vocabulary sitting in a functional note that should have been pulled into a constraint. This skill sequences and reconciles; it doesn't own any layer's artifact shape or proposal logic itself.

## When to use

- The user describes a product and wants it modeled comprehensively, not one layer at a time: "model this whole product", "map out the full picture for a habit tracker", "I want the data model, jobs, performers, and workflows all set up."
- The user has already modeled some layers and asks to fill in the rest: still trigger this skill, but see step 0 — it skips layers that are already substantially populated rather than re-proposing them.
- The user has already run `speck` (or modeled all five layers manually) and wants it reconciled or double-checked: trigger this skill again — step 0 will find the model already populated and skip straight to step 6's second pass.

Do **not** trigger this for single-layer requests ("what entities would this need?", "let's define the performers") — those go to the specific `model-*` skill, which is more precisely scoped for that conversation.

## Workflow

### 0. Identify the vault once, and detect what's already there

Delegate to [[create-vault]] with no path (unless the user named one) to resolve a single vault path for this whole run. Every subsequent skill call below reuses that same path — none of them should re-run vault discovery.

If the resolved vault already has notes in one or more of `entities/`, `jobs/`, `performers/`, `workflows/`, tell the user which layers already have content and confirm whether to skip those and only run the empty ones, or re-propose additions on top of what's there. Don't silently skip or silently re-propose — ask.

Check `constraints/` too, but treat it differently: report what's there if anything, but an empty `constraints/` folder is not a gap the way an empty `entities/` folder would be — see step 5.

If entities, jobs, performers, and workflows all already have notes, there's nothing left to propose for those four — confirm with the user, then skip straight to step 5. This is how re-running `speck` on an already-modeled vault turns into a reconciliation pass instead of redoing work.

### 1. Model entities

Delegate to [[model-entities]], passing the resolved vault path (skip its own vault-discovery step 1 entirely). Propose the entity set, confirm with the user, write each one via `create-entity`.

Do not proceed to step 2 until this layer is confirmed and written — jobs reference these entities.

### 2. Model jobs

Delegate to [[model-jobs]], passing the resolved vault path. Propose the job set — referencing the entities just confirmed — confirm with the user, write each one via `create-job`.

### 3. Model performers

Delegate to [[model-performers]], passing the resolved vault path. Propose performers for the confirmed jobs, confirm, write each one via `create-performer`.

### 4. Model workflows

Delegate to [[model-workflows]], passing the resolved vault path. Propose workflows for the confirmed jobs, performers, and entities, confirm, write each one via `create-workflow`.

### 5. Check for constraints

Delegate to [[model-constraints]], passing the resolved vault path. This step is structurally different from the previous four — don't propose a default constraint set the way entities or jobs get proposed. `model-constraints` exists specifically because constraints get captured reactively, one at a time, when a real implementation decision is made — not brainstormed in bulk (see its own "Why this exists").

Instead: check whether describing this product surfaced any real implementation commitment — a deployment mode ("this needs to run on-prem too"), an auth requirement ("has to delegate to the customer's own IDP"), a protocol choice ("AI clients only talk to this via MCP"), or similar. If it did, propose those specific constraints, confirm, and write them via `create-constraint`. If nothing implementation-specific came up in the conversation, say so plainly and move on — an empty `constraints/` folder at this point is the expected, healthy state, not something to fill preemptively.

### 6. Second pass: refine using what the full model now reveals

Don't skip this step or treat it as optional busywork. Each `model-*` skill's own "Show the user what was created" step already defines a cross-check against its upstream layers, but on a single forward pass each check only ever runs once, immediately after its own layer, as a suggestion the user may never act on — it never gets revisited with what the *later* layers went on to reveal. This step is where that gets reconciled, using checks that already exist, not new logic invented here.

Now that the model exists, walk all five layers again in the same order, but scoped only to what each check flags — this is a targeted patch pass, not a second full proposal round:

1. **Entities** — per [[model-jobs]]'s cross-check: any entity no job touches? Per [[model-workflows]]'s cross-check: any entity field a step's Transition needed that the Fields table doesn't have?
2. **Jobs** — per [[model-performers]]'s cross-check: any job with no performer executing it? Per [[model-workflows]]'s "Show what was created" step: any job whose `## Tasks` sketch stayed thin despite a workflow now elaborating it?
3. **Performers** — any performer whose `main_job` turned out to need splitting once a workflow made its steps concrete (two genuinely different executors hiding behind one job)?
4. **Workflows** — any job or performer that clearly needs a workflow and still has none?
5. **Constraints** — per [[model-constraints]]'s cross-check: does any entity, job, performer, workflow, or surface note's prose use implementation vocabulary (deployment, protocol, auth mechanism, database, hashing, transport) that should have been pulled out into a constraint instead? Does any existing constraint's `applies_to` reference a note that doesn't exist, or duplicate ground another constraint already covers?

Present the findings as one consolidated list grouped by layer, and confirm with the user before applying any of it — a flagged gap isn't automatically a bug. A support job genuinely having no performer, plumbing genuinely having no job, or a product genuinely having no implementation commitments worth constraining yet, are valid outcomes the checks will still flag; don't "fix" what the user confirms is intentional. If skip-confirmation mode is active (see Rules below), still present the full findings list first — that part never goes silent — then proceed straight to applying the fixes instead of waiting for a reply, the same show-then-act rule the forward pass uses in each layer's proposal step.

For each confirmed refinement, delegate back to the specific skill that owns that layer — add a field via [[create-entity]] (editing the existing note, not rewriting it), add a job via [[create-job]], pull implementation vocabulary out into a new note via [[create-constraint]], and so on. Run this pass once; if the user wants to iterate further after seeing the result, they can just run `speck` again rather than looping automatically here.

### 7. Report

Summarize what was created across all five layers — counts per kind and the vault path — plus anything skipped in step 0 and anything refined in step 6. If step 5 found no constraints worth capturing, say that plainly rather than omitting the layer from the summary.

## Rules

- **Never collapse the confirmations by default.** Each layer, and the second pass, proposes before writing unless the user has explicitly opted into skip-confirmation mode — see the Skip-confirmation mode note in each `model-*` skill's proposal step. This skill doesn't define its own version of that opt-out; it passes the user's instruction through to whichever layer it delegates to next, and applies the same rule to its own step 6.
- **An unscoped opt-out is global; a scoped one isn't.** "Don't ask for confirmation" with no layer named applies to every remaining call in this run, through step 6. "Skip it for jobs" applies only to the jobs layer — every other layer still proposes and waits.
- **Skip-confirmation mode changes when writing happens, not what gets shown.** Steps 1–5's proposals (or, for step 5, its finding of nothing to propose) and step 6's findings are always presented first; the mode only removes the blocking wait for a reply before acting on them. It never goes silent until the final report.
- **Respect a mid-pipeline stop, independent of skip-confirmation mode.** If the user says "just entities and jobs for now" — during step 0 or after any layer completes — stop there instead of forcing every layer and the second pass. Skipping the wait on each layer's proposal says nothing about how many layers to run; they're separate instructions and don't imply each other.
- **A flagged gap is a question, not an automatic edit — surfaced the same way under either mode.** The second pass always shows its findings before touching anything. Under the default it then waits for confirmation; under skip-confirmation mode it proceeds straight to applying them. Either way, don't silently delete orphan entities, silently invent a performer to fill a hole, or silently invent a constraint nothing actually constrains, without the finding having been visible first.
- **Don't rewrite any layer's artifact shape, proposal heuristics, or cross-check logic inline here.** Those belong to `model-entities` / `model-jobs` / `model-performers` / `model-workflows` / `model-constraints` respectively — this skill only re-invokes them at a point in time (after the full model exists) when their checks have more to work with.
- **Constraints don't get a forward-pass default.** Steps 1–4 propose real content because a product almost always has entities, jobs, performers, and workflows worth sketching immediately. Step 5 is different by design — see its own text — and finding nothing to propose there is a normal outcome, not a skipped step.
