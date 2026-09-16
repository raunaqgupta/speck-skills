# Speck skills

A collection of skills that provide interconnected building blocks to design a product: jobs, performers, workflows, constraints, entities, and surfaces.

## Installation

This repo is packaged as a **Claude Code plugin** (`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` at the root). That's the fully-supported install path. Claude Desktop can run it too, via the same plugin marketplace — see caveats below.

### Claude Code (CLI or IDE extension)

1. Add this repo as a plugin marketplace:
   ```
   /plugin marketplace add raunaqgupta/speck-skills
   ```
   This clones the repo and registers a marketplace named `speck` (the `name` field in `marketplace.json`, not the GitHub repo name — they differ).
2. Install the plugin from that marketplace:
   ```
   /plugin install speck@speck
   ```
   (`<plugin-name>@<marketplace-name>`, both `speck` for this repo's manifests.)
3. Verify it installed and see all 14 skills registered:
   ```
   /plugin list
   ```
4. Skills trigger automatically in conversation once installed — no further setup. Restart Claude Code if a skill doesn't seem to be picked up.

To update later: `/plugin marketplace update` pulls the latest commit, then reinstall if a new version is pinned.

### Claude Desktop

Claude Desktop supports plugins two ways, and this repo only cleanly supports one of them:

- **Plugin marketplace (works as-is).** Add this repo as a marketplace via **Plugins & skills → Add marketplace** (choose GitHub repo or Git URL, pointing at `raunaqgupta/speck-skills`), then install the plugin from it — any individual user can do this from the app, no admin required. An admin can additionally provision it org-wide via the `allowedPluginMarketplaces` managed-config key, which pins it centrally and auto-installs it for everyone; if both exist, the admin-configured entry takes precedence. Either way it uses the same `marketplace.json` as the Claude Code path above, so no repackaging needed.
- **Per-user skill upload (needs rework first — don't attempt as-is).** Claude.ai/Desktop's own **Settings → Capabilities → Skills** flow lets you zip an individual skill folder and upload it, but it caps the `description` frontmatter field at 200 characters. Every `SKILL.md` in this repo is written for Claude Code, which has no such cap — descriptions here run 600–1,100 characters, tuned for precise auto-triggering. Uploading `skills/<name>/` as-is will fail Claude.ai's validation. Shortening eight descriptions to fit would lose most of the triggering nuance, so this isn't recommended over the marketplace path above.

## Usage

Once installed, use naturally in conversation:

- "I want to build a habit tracker — let's model the whole thing." (entities, jobs, performers, and workflows, in sequence)
- "I want to build a habit tracker — let's model the entities." (just this one layer)
- "What job is this product doing for the user?"
- "Who actually performs this job?"
- "Walk through the steps of the login flow as a workflow."
- "Add a Notification entity to the vault at ~/projects/my-app."

Speck will propose a set, confirm with you, then write the notes.

## Skills

**`speck`** — Models a whole product in one command, in two passes: first entities, then jobs, then performers, then surfaces, then workflows, then a check for constraints, each layer delegated to its own skill below in dependency order, with a propose-and-confirm round per layer; then a second pass that applies each skill's own cross-checks against the now-complete model — catching gaps like a missing entity field, a thin job sketch that only became visible once workflows made things concrete, a job reachable two ways with only one surface covering it, or implementation vocabulary stranded in a functional note instead of pulled into a constraint. Use for "model this whole product" requests; use a single `model-*` skill directly when you only want one layer.

**`model-entities`** — Model the core nouns of a product (User, Task, Project, etc.) as linked entity notes in an Obsidian vault. Each note describes fields, relations, and invariants. Relations use `[[wiki links]]` so the vault becomes a clickable, graph-viewable schema.

**`model-jobs`** — Model the jobs users hire the product to do, using the Christensen JTBD framework: situation, outcome, the four forces (push, pull, habit, anxiety), and the entities each job touches. Jobs link to entities so the same vault holds both _what_ the product is and _why_ anyone would use it.

**`model-performers`** — Model the functional roles who execute those jobs — the performer, defined by the act and explicitly separated from adjacent roles (buyer, approver, reviewer), not by demographics.

**`model-surfaces`** — Model the product's delivery surfaces (a web app, an API, an MCP server) as boundary notes — what each is and isn't responsible for, and which jobs it delivers to which performers. Like entities/jobs/performers/workflows and unlike constraints, a product's surfaces are a small, fixed set proposed together, early — needed before workflows, since every workflow note cites the surface it happens on.

**`model-workflows`** — Model the concrete interaction sequences (steps, screens, entity-state transitions) that carry out a job — or that stand alone with no job behind them, like routine login. This is the *how*, kept separate from the job's *why*.

**`model-constraints`** — Model implementation commitments (deployment modes, protocol choices, auth mechanisms) as constraint notes, each linked via `applies_to` to the functional notes it binds — keeping that vocabulary out of the entity/job/performer/surface/workflow notes themselves. Unlike the other five layers, constraints are captured reactively, one or a few at a time as real decisions get made, not brainstormed in bulk.

**`create-entity`** / **`create-job`** / **`create-performer`** / **`create-surface`** / **`create-workflow`** / **`create-constraint`** — Primitives for writing a single note of each kind. Called by the modeling skills above, or directly when adding one artifact to an existing vault.

**`create-vault`** — Resolves or creates the vault every other skill depends on: finds an already-open, user-named, in-repo, or pre-existing standalone vault, or bootstraps a new one (the six kind folders, all `Templates/*.md` files, `.obsidian/templates.json`, `.obsidian/graph.json` color groups). The single place this logic lives — every other skill delegates here rather than to each other, so none of them implicitly depends on another having run first. Idempotent, and called automatically before any skill's first write — you normally never invoke it directly.
