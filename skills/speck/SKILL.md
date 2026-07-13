---
name: speck
description: Model an entire product end-to-end in one command — entities, jobs, performers, and workflows — by sequencing the model-entities, model-jobs, model-performers, and model-workflows skills in dependency order against a single vault. Use when the user wants the full picture in one go rather than one layer at a time — "model this whole product", "let's fully map this app out", "set up the complete model for X: data, jobs, performers, and workflows", "I want everything mapped, not just the entities". If the user only wants a single layer ("just the entities", "what jobs would this need?"), trigger that layer's own model-* skill directly instead — this skill is specifically for requests that span multiple or all layers at once.
---

# Speck

Runs the full modeling pipeline against one vault: entities, then jobs, then performers, then workflows — each layer delegated to its own skill, in the dependency order that lets later layers reference earlier ones (jobs touch entities, performers execute jobs, workflows carry out jobs via performers and touch entities). This skill sequences; it doesn't own any layer's artifact shape or proposal logic itself.

## When to use

- The user describes a product and wants it modeled comprehensively, not one layer at a time: "model this whole product", "map out the full picture for a habit tracker", "I want the data model, jobs, performers, and workflows all set up."
- The user has already modeled some layers and asks to fill in the rest: still trigger this skill, but see step 0 — it skips layers that are already substantially populated rather than re-proposing them.

Do **not** trigger this for single-layer requests ("what entities would this need?", "let's define the performers") — those go to the specific `model-*` skill, which is more precisely scoped for that conversation.

## Workflow

### 0. Identify the vault once, and detect what's already there

Delegate to [[create-vault]] with no path (unless the user named one) to resolve a single vault path for this whole run. Every subsequent skill call below reuses that same path — none of them should re-run vault discovery.

If the resolved vault already has notes in one or more of `entities/`, `jobs/`, `performers/`, `workflows/`, tell the user which layers already have content and confirm whether to skip those and only run the empty ones, or re-propose additions on top of what's there. Don't silently skip or silently re-propose — ask.

### 1. Model entities

Delegate to [[model-entities]], passing the resolved vault path (skip its own vault-discovery step 1 entirely). Propose the entity set, confirm with the user, write each one via `create-entity`.

Do not proceed to step 2 until this layer is confirmed and written — jobs reference these entities.

### 2. Model jobs

Delegate to [[model-jobs]], passing the resolved vault path. Propose the job set — referencing the entities just confirmed — confirm with the user, write each one via `create-job`.

### 3. Model performers

Delegate to [[model-performers]], passing the resolved vault path. Propose performers for the confirmed jobs, confirm, write each one via `create-performer`.

### 4. Model workflows

Delegate to [[model-workflows]], passing the resolved vault path. Propose workflows for the confirmed jobs, performers, and entities, confirm, write each one via `create-workflow`.

### 5. Report

Summarize what was created across all four layers — counts per kind and the vault path — plus anything skipped in step 0.

## Rules

- **Never collapse the confirmations.** Each layer still gets its own propose-then-confirm round from its underlying skill. This skill sequences four conversations, it doesn't replace them with one unreviewed dump of every note across every kind.
- **Respect a mid-pipeline stop.** If the user says "just entities and jobs for now" — during step 0 or after any layer completes — stop there instead of forcing all four.
- **Don't rewrite any layer's artifact shape or proposal heuristics inline here.** Those belong to `model-entities` / `model-jobs` / `model-performers` / `model-workflows` respectively.
