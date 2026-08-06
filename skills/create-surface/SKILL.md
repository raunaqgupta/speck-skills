---
name: create-surface
description: Write a single surface note into an Obsidian vault's `surfaces/` folder, using the canonical surface shape (frontmatter `tags: [surface]`, `kind: product` or `service`, optional `consumes`/`consumed_by` links, a prose boundary description, a Not Responsible For section, and a Provides section naming the jobs it delivers to which performers). Use when the user wants to add one delivery surface to an existing vault — "add a surface for the mobile app", "define the CLI as a surface", "what's the API's boundary as a surface" — and as the file-writing primitive that the `model-surfaces` skill delegates to once a confirmed surface set has been agreed. This skill owns the artifact shape; the parent `model-surfaces` skill owns the conversation and the multi-surface proposal. A surface is a functional boundary — what it's responsible for and who it serves — never an implementation commitment (that's a constraint's job).
---

# Create Surface

Writes one surface note to `<vault>/surfaces/<SurfaceName>.md` with the canonical shape used across this design system. Self-contained: can be invoked directly by a user (`/create-surface "<Name>"`) or delegated to by `model-surfaces` once a confirmed surface has been chosen.

## When to use

- The user names a single delivery surface to define: "add the mobile app as a surface", "define the CLI's boundary".
- A parent skill (`model-surfaces`) has confirmed a surface set and needs to write each file.
- The user is iterating in an existing vault and wants one more surface stood up quickly.

Do **not** trigger from cold context where the user hasn't yet identified a vault or agreed on the surface set — that's `model-surfaces`'s job. This skill assumes those decisions are made.

Do **not** use this for implementation commitments (deployment mode, protocol choice, auth mechanism) — that's a [[create-constraint]] note bound to this surface via `applies_to`, not part of the surface note itself. A surface note stays implementation-free, same as entity/job/performer/workflow notes.

## Inputs

The caller (user or parent skill) provides:

1. **Vault path** — absolute path to the Obsidian vault root. If invoked directly without one, delegate to [[create-vault]] to resolve or create it.
2. **Surface name** — a short noun phrase naming the delivery surface: `Speck Web App`, `MCP Server`, `Mobile App`. Used as the filename.
3. **Kind** — `product` (serves the end user directly, human-facing) or `service` (serves other clients or surfaces — an API, an integration point).
4. **Boundary description** — 2–3 sentences: what this surface is responsible for, and — if it has consumers other than the end user — the job it does for them.
5. **Not responsible for** — 1 or more bullets naming adjacent concerns this surface explicitly does *not* own, each with a one-line pointer to whichever surface or note does own it. Every surface needs at least one of these once a second surface exists to be confused with — a lone surface in a vault may have nothing to exclude yet.
6. **Provides** — 1 or more bullets, each `[[Job]] — for <performer>`, naming which jobs this surface delivers to which performer. A surface with nothing here isn't providing anything yet — check whether it's actually needed.
7. *(Optional)* **Consumes** / **Consumed by** — `[[wiki links]]` to other surfaces this one calls, or is called by. Omit both if this surface only serves end users directly.

If any required input is missing, ask the user before writing.

## Workflow

### 1. Ensure vault prerequisites

Delegate to [[create-vault]] with this vault path. It ensures `<vault>/surfaces/` exists, `<vault>/Templates/Surface.md` exists (writing it from its canonical template if missing, never overwriting an existing one), `<vault>/.obsidian/templates.json` points at `Templates/`, and `<vault>/.obsidian/graph.json` has a color group for the `surface` tag. It's idempotent — safe to call even if the vault is already fully set up. **Do not duplicate that setup logic here.**

### 2. Write the surface file

Write `<vault>/surfaces/<SurfaceName>.md` with this shape:

```markdown
---
tags:
  - surface
kind: product
consumes:
  - "[[ ]]"
consumed_by:
  - "[[ ]]"
---

<2–3 sentences: what this surface is responsible for — its boundary — and, if it has consumers other than the end user, the job it does for them.>

## Not responsible for

- **<Adjacent concern>** — owned by [[OtherSurface]] instead.

## Provides

- [[Job sentence]] — for <performer>
```

Rules for filling it in:

- **`tags`** is always `[surface]`. Never omit.
- **`kind`** — `product` or `service`. A surface with no other clients calling it is almost always `product`; a surface other surfaces or AI clients call into is `service`.
- **`consumes`** / **`consumed_by`** — omit either key entirely if empty, not `consumes: []`. Most surfaces in a small product have neither.
- **Filename** — the surface's own name, title-case noun phrase: `Speck Web App.md`, `MCP Server.md`. Not a generic label like `Web Surface.md`.
- **Boundary description** — a functional boundary, not an implementation commitment. "The AI-facing integration surface" is a boundary; "runs on Fastify over WebSockets" is an implementation detail that belongs in `code/` or a constraint bound to this note, not the boundary prose itself.
- **Not responsible for** — only list concerns a reader might actually attribute to this surface by mistake. Point at the surface or note that actually owns each one.
- **Provides** — every entry names a real job (`[[wiki link]]`) and the performer it's delivered to. Don't invent a job here — if the job doesn't exist yet, note it as unresolved and flag it as a candidate for `model-jobs`.
- **Don't overwrite** — if the file already exists, stop and ask the user whether to replace, merge, or skip.

### 3. Report

Print the path written, the jobs it provides (and to which performers), what it's explicitly not responsible for, and any unresolved `[[wikilinks]]` (jobs or performers that don't yet exist — candidates for `model-jobs` or `model-performers`).

The canonical `Templates/Surface.md` body lives in [[create-vault]]'s Reference section — it's the single source of truth, so it isn't duplicated here.
