---
name: jtbd-modeling
description: Model the jobs-to-be-done (JTBD) of a product — the outcomes users hire it for, the situations that trigger them, the steps they walk through, and the four forces (push, pull, habit, anxiety) acting on the switch — by creating one Markdown note per job inside an Obsidian vault's `jobs/` folder. Use this skill whenever the user is brainstorming, scoping, or planning a product and the conversation touches on user motivation, outcomes, switch moments, "why people would use this," moments-of-use, customer interviews, or the JTBD framework explicitly. Trigger even if the user doesn't say "JTBD" — phrases like "what job does this do for the user?", "why would they switch?", "what's the moment they reach for this?", "model the user's goal", "what are people trying to accomplish?", or any conversation framing the product around user outcomes rather than features. Pairs with the entity-modeling skill: jobs reference entities via `[[wiki links]]` so the same vault holds both the data model and the motivation model.
---

# JTBD Modeling

This skill turns product-planning conversations into a navigable set of job notes in an Obsidian vault. Each note describes one job-to-be-done: the situation that triggers it, the outcome the user is hiring the product for, the steps they walk through, and the four forces tugging at the switch. Jobs link to the [[entities]] they touch, so the vault holds both *what* the product is (entities) and *why* anyone would use it (jobs) in one graph.

## Why this exists

A product's entity model tells you what it stores; its job model tells you what it's *for*. Without the job layer, feature decisions drift toward whatever's easy to build. With it, every entity, screen, and endpoint can be traced back to a user outcome someone is willing to switch for. Doing this in Obsidian — alongside the entities — means the two layers stay linked, and the graph view shows which entities serve which jobs.

## When to use

Trigger this skill when the user is **planning or validating** a product and the conversation turns to user motivation. Examples:

- "What job is this product really doing for the user?"
- "Why would someone switch to this from what they use now?"
- "Let's map out the moments people would reach for this."
- "I want to do a JTBD breakdown of this idea."
- The user has named entities and now needs to ground them in outcomes.

Do **not** trigger when the user wants user personas (different framework), feature lists, or competitive analysis. JTBD is about *jobs people hire products to do*, not about who the people are.

## Workflow

### 1. Identify the vault

Same as the [[entity-modeling]] skill. If a vault already exists for this product (likely, since jobs usually come after some entity work), use it. Otherwise see `entity-modeling`'s step 1 for the vault-discovery flow.

### 2. Propose the job set before writing

Read the product description and draft 3–7 candidate jobs. Lean toward fewer, sharper jobs over a long flat list — most products serve one or two **main jobs** plus a small set of **related jobs**. Common patterns:

- **The headline job** — the single sentence that, if a stranger heard it, would make them say "oh, I'd use that."
- **Recurring / habitual jobs** — what someone does every day or every week with the product.
- **Episodic jobs** — rare but high-stakes moments (onboarding a teammate, recovering from a mistake, year-end review).
- **Emotional jobs** — jobs that are really about how the user wants to *feel* (calm, in control, prepared).

For each candidate, draft a one-line job statement in the Christensen form: **"When [situation], I want to [motivation], so I can [outcome]."** Present the list to the user, with the entities each job touches in parentheses. Confirm before writing files. Like entity-modeling, this is the single most valuable interaction in the skill.

### 3. Create one job at a time via `/create-job`

For each confirmed job, delegate the file-writing to the [[create-job]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [job]` + `situation` + `outcome` + `touches` + the four `forces`, prose description, numbered Tasks list), handles vault prerequisites (`jobs/` folder, `Templates/Job.md`, `.obsidian/templates.json`), and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per job, passing:

- Vault path
- Job sentence (sentence-cased verb phrase: `Plan my week`, never `PlanMyWeek`)
- Concrete `situation` and user-POV `outcome`
- Entities touched, grouped by `reads` / `creates` / `updates`
- All four forces (`push`, `pull`, `habit`, `anxiety`) — write `"(none identified — revisit after user interviews)"` for any that genuinely can't be named, rather than dropping the kind
- Prose description (2–4 sentences)
- Numbered Tasks list with the entity each task touches
- Optional success criteria

Recommended order: write the **headline job** first, then habitual jobs, then episodic and emotional ones. This catches naming-convention drift early and makes the rest of the set easier to write.

If the user adds a job mid-flow ("also add one for offboarding"), just call `/create-job` once more. The separation exists for exactly this case.

### 4. Show the user what was created

List the files. Offer next steps:

- Add jobs they think are missing (especially episodic or emotional ones, which tend to get forgotten)
- Drill into any job to flesh out tasks or sharpen the forces
- Cross-check: every entity should be touched by at least one job. Entities no job needs are candidates for deletion.
- Move on to UI screens or API endpoints — each can cite the job(s) it serves.

## Style notes

- One job per file. Resist the urge to bundle "morning planning" and "evening review" into one note — they're different jobs with different forces.
- Sentence-cased verb phrases for filenames (`Plan my week`, `Capture an idea`) — never PascalCase, never noun-led. The filename should read like the user's own words for the job.
- Link liberally with `[[wiki links]]`. Links to entities that don't exist yet are fine — they appear as unresolved links and signal "we should model this next."
- Don't invent forces the user hasn't implied. If you can't name a credible anxiety, write "(none identified — revisit after user interviews)" rather than fabricating one.
- Don't write implementation, UI mockups, or feature lists. This skill stops at user-outcome modeling.
- JTBD uses "task" for sub-actions inside a job. This is the framework's term — keep it even when the product being modeled has an entity also called `Task`. The two live in different folders (`jobs/` vs `entities/`) and disambiguate by context.
