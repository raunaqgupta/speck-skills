---
name: model-surfaces
description: Model the delivery surfaces of a product — the distinct human- or client-facing boundaries through which the product is actually reached (a web app, a CLI, an API, an MCP server) — by creating one Markdown note per surface inside an Obsidian vault's `surfaces/` folder. Each note states what that surface is and isn't responsible for and which jobs it delivers to which performers, keeping implementation vocabulary (deployment, protocol, hosting) out — that belongs in a constraint bound to the surface instead. Use this skill when the user is scoping how many distinct ways the product will actually be delivered or accessed: "we'll have a web app and an API", "what are the surfaces here?", "does the CLI count as its own surface?". Like entities/jobs/performers/workflows and unlike constraints, a product's surfaces are usually a small, fixed set decided together, early — propose them as a batch, not one at a time. Needs jobs and performers to already exist (a surface's Provides section names both); needed before workflows are modeled (every workflow note links to the surface it belongs to).
---

# Surface Modeling

This skill turns product-delivery conversations into a navigable set of surface notes in an Obsidian vault. Each note names one functional boundary — a web app, an API, an MCP server, a CLI — states what it's responsible for and explicitly not responsible for, and lists which jobs it delivers to which performers. Surfaces are what let a workflow say "this happens on the web app" without re-explaining what the web app is every time, and what let the model represent "the same job, reachable two different ways" (a Collaborator adding a Node through the web UI or through their own AI session) as two workflows sharing one job but citing different surfaces.

## Why this exists

A product is rarely delivered through exactly one door. Even a simple tool might have a web app and a public API; an AI-native one might have a human-facing UI and an MCP server AI clients call into. Without a surface layer, that split either goes undocumented (every workflow silently assumes "the app" without saying which one) or gets smuggled into system/architecture docs that mix functional boundaries with implementation detail. Surface notes hold the functional half — what each door is for and who walks through it — while constraints (see [[model-constraints]]) hold the implementation half — how it's actually built, deployed, or secured.

Surfaces also give "not responsible for" a real home. The single most common source of scope confusion in a growing product is two surfaces quietly overlapping — the web app growing an ad-hoc API, the API growing a UI. Naming each surface's boundary explicitly, and what it defers to its neighbor, is what keeps that from happening silently.

## When to use

Trigger this skill when the user is scoping how the product is actually delivered or reached. Examples:

- "We need a web app and a public API."
- "What are the surfaces for this product?"
- "Does the mobile app count as its own surface, or is it just the web app in a wrapper?"
- "Who's responsible for X — the app or the API?"
- Jobs and performers exist, and workflows are about to be modeled but nothing says which surface each one happens on.

Do **not** trigger to brainstorm surfaces speculatively before any jobs exist — a surface's `## Provides` section names real jobs and performers; with nothing to provide yet, there's nothing to propose. Offer to run [[model-jobs]] and [[model-performers]] first if they're missing. Do **not** use this for implementation commitments (deployment mode, protocol, hosting) — those are constraints bound to a surface via `applies_to`, not part of the surface note itself; see [[model-constraints]].

## Workflow

### 1. Identify the vault

Delegate to [[create-vault]] with no path (unless the user named one). It resolves an already-open or already-resolved vault, finds an existing in-repo or standalone vault, or proposes and creates a new one. Don't duplicate that logic here, and don't look to any other `model-*` skill for it — `create-vault` is the one place it's written.

### 2. Check for existing jobs and performers

Read the vault's `jobs/` and `performers/` folders. A surface's `## Provides` section exists to name real jobs delivered to real performers — if neither folder has content yet, surfaces have nothing to reference. Offer to run [[model-jobs]] and [[model-performers]] first rather than proposing surfaces with empty or speculative Provides sections.

### 3. Propose the surface set before writing

Read the confirmed jobs and performers and draft one surface per distinct delivery boundary. A few rules:

- **Default to few.** Most products have one or two surfaces (a web app; a web app plus an API). More than three is unusual — check whether some are really the same surface reached different ways rather than genuinely separate boundaries.
- **Split on audience and protocol, not on screen or feature.** "The web app" is one surface even if it has many pages; "the API" is a different surface because a fundamentally different kind of client reaches it, not because it does different things.
- **Every job that has more than one performer path likely implies more than one surface.** If a job is done by a human through a UI *and* by an AI session through an API, that's the same job reachable via two surfaces — model both surfaces, and let the (later) workflows for that job cite whichever surface fits each path.
- **Name what each surface is NOT responsible for, relative to its siblings.** This is the check that catches quiet scope overlap — for every pair of surfaces, ask what happens if a capability could plausibly live on either, and pin it to one explicitly.

For each candidate, present:
- Surface name
- Kind (`product` — human-facing, or `service` — called by other clients/surfaces)
- The jobs it provides, and to which performers
- What it's explicitly not responsible for (relative to any sibling surfaces)

Present the list to the user and confirm before writing files.

**Skip-confirmation mode.** If the user has explicitly said something like "don't ask for confirmation, just write it" or "stop confirming with me," skip the wait: still show the proposed list above, then move straight to writing instead of pausing for a reply. Unless they scope it narrower, treat it as a standing preference for the rest of the session — covering every `model-*` skill and `speck` call from here on. Never infer this from a fast or approving reply; it has to be requested explicitly.

### 4. Create one surface at a time via `/create-surface`

For each confirmed surface, delegate the file-writing to the [[create-surface]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [surface]` + `kind` + optional `consumes`/`consumed_by`, boundary prose, Not Responsible For, Provides), delegates vault prerequisites (`surfaces/` folder, `Templates/Surface.md`, `.obsidian/templates.json`, graph color group) to [[create-vault]], and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per surface, passing:

- Vault path
- Surface name and kind (`product`/`service`)
- Boundary description (2–3 sentences)
- Not responsible for (adjacent concerns, each pointing at the surface or note that owns it)
- Provides (jobs delivered, and to which performers)
- Consumes/consumed by, if this surface calls or is called by another

### 5. Show the user what was created

List the files. Offer next steps:

- Model the workflows that happen on each surface next via [[model-workflows]] — every workflow note cites the surface it belongs to, so surfaces should exist before workflows do.
- Cross-check: does every job with more than one performer path have a surface for each path? A job only ever cited from one surface's Provides section, when the model shows two performers reaching it differently, is a sign a surface is missing.
- Cross-check: do any two surfaces' Not Responsible For sections leave a gap — a capability neither claims and neither explicitly defers?

## Style notes

- **One surface per file, one boundary per surface.** If you're tempted to combine two, ask whether they're reached by the same kind of client through the same protocol. If not, they're separate.
- **Boundary prose, not build detail.** "The AI-facing integration surface" is a boundary. "Runs on Fastify, authenticated with bearer tokens" is a constraint — write it as one, bound to this surface via `applies_to`, not folded into the surface note.
- **Provides is a real list, not a formality.** Every entry should be a job that genuinely reaches this surface. An empty Provides section is a sign the surface isn't earning its place yet.
- **Not Responsible For is where scope disputes get resolved before they happen.** Write it defensively — name the thing a reader would most plausibly misattribute to this surface.
- **Don't front-run workflows.** Surfaces say *where* something happens, not the steps of *how* — that's [[model-workflows]]'s layer, modeled next.
