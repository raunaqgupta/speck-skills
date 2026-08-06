---
name: model-jobs
description: Model the jobs-to-be-done (JTBD) of a product — the outcomes users hire it for, the situations that trigger them, the steps they walk through, and the four forces (push, pull, habit, anxiety) acting on the switch — by creating one Markdown note per job inside an Obsidian vault's `jobs/` folder. Use this skill whenever the user is brainstorming, scoping, or planning a product and the conversation touches on user motivation, outcomes, switch moments, "why people would use this," moments-of-use, customer interviews, or the JTBD framework explicitly. Trigger even if the user doesn't say "JTBD" — phrases like "what job does this do for the user?", "why would they switch?", "what's the moment they reach for this?", "model the user's goal", "what are people trying to accomplish?", or any conversation framing the product around user outcomes rather than features. Pairs with the model-entities and model-performers skills: jobs reference entities via `[[wiki links]]` and performers link to the jobs they execute, so the same vault holds the data model, the motivation model, and the people model.
---

# Job Modeling

This skill turns product-planning conversations into a navigable set of job notes in an Obsidian vault. Each note describes one job-to-be-done: the situation that triggers it, the outcome the user is hiring the product for, the steps they walk through, and the four forces tugging at the switch. Jobs link to the [[entities]] they touch, so the vault holds both *what* the product is (entities) and *why* anyone would use it (jobs) in one graph.

## Why this exists

A product's entity model tells you what it stores; its job model tells you what it's *for*. Without the job layer, feature decisions drift toward whatever's easy to build. With it, every entity, screen, and endpoint can be traced back to a user outcome someone is willing to switch for. Doing this in Obsidian — alongside the entities and performers — means the three layers stay linked, and the graph view shows which entities serve which jobs and which performers execute which jobs.

## When to use

Trigger this skill when the user is **planning or validating** a product and the conversation turns to user motivation. Examples:

- "What job is this product really doing for the user?"
- "Why would someone switch to this from what they use now?"
- "Let's map out the moments people would reach for this."
- "I want to do a JTBD breakdown of this idea."
- The user has named entities and now needs to ground them in outcomes.

Do **not** trigger when the user wants to identify who executes the jobs (use [[model-performers]] for that), feature lists, or competitive analysis. JTBD is about *jobs people hire products to do*, not about who the people are. Do **not** require entities to exist before modeling jobs — check for them, but proceed without it if the user wants to sketch jobs speculatively (see step 2).

## Workflow

### 1. Identify the vault

Delegate to [[create-vault]] with no path (unless the user named one). It resolves an already-open or already-resolved vault, finds an existing in-repo or standalone vault, or proposes and creates a new one. Don't duplicate that logic here, and don't look to `model-entities` or any other `model-*` skill for it — `create-vault` is the one place it's written.

### 2. Check for existing entities

Before proposing jobs, check whether the vault has an `entities/` folder with entity notes. Jobs read/create/update entities via `touches` — if entities don't exist yet, offer to run [[model-entities]] first. Proceed without them only if the user wants to sketch jobs speculatively; flag which entities each job implies as unresolved links.

### 3. Propose the job set before writing

Read the product description and draft 3–7 candidate jobs. Lean toward fewer, sharper jobs over a long flat list — most products serve one or two **main jobs** plus a small set of **related jobs**. Common patterns:

- **The headline job** — the single sentence that, if a stranger heard it, would make them say "oh, I'd use that."
- **Recurring / habitual jobs** — what someone does every day or every week with the product.
- **Episodic jobs** — rare but high-stakes moments (onboarding a teammate, recovering from a mistake, year-end review).
- **Emotional jobs** — jobs that are really about how the user wants to *feel* (calm, in control, prepared).

For each candidate, draft a one-line job statement in the Christensen form: **"When [situation], I want to [motivation], so I can [outcome]."**

**Before presenting the list, screen every candidate — not just the headline — against these checks.** This is a self-check pass over your own draft, not a mechanical gate: a genuinely thin-but-real job (a niche episodic job with only two clear forces, say) should still make the list if you judge it's real. The point is catching a specific failure mode — a capability or migration requirement dressed up in JTBD scaffolding — not raising the bar on job quality generally.

- **Stranger test, applied to every candidate, not just the headline.** If a stranger heard this job stated plainly, would they say "oh, I'd use that" — or would you first have to explain the product's own mechanism before the "job" makes sense? A candidate that only lands once you already understand how the product works internally has failed this test.
- **One level deeper than the mechanism.** Christensen's own example: nobody hires a drill for its own sake — they hire it to put a hole in the wall, and the hole is really in service of a hung picture. Check whether your draft stops at restating the product's own feature or mechanism (e.g. "collaborate live, no file conflicts") rather than naming the human outcome one level underneath it. If it reads like a feature-list entry with "I want to" bolted on, push it down a level.
- **`outcome` should name a state achieved, not an absence.** "Nothing changes for the user," "no disruption to my workflow," "I don't have to give anything up" — these describe the absence of friction during a migration, not a state someone is hiring the product to reach. If the outcome you've drafted only negates something rather than naming what's now true, treat that as a sign the candidate is a requirement wearing job scaffolding.
- **Hollow or circular forces are a tell, not just a gap to note.** Rough out all four forces for each candidate before finalizing the list (see step 4 for the full force-writing rules). If two or more come back "(none identified)" or reduce to circular restatements of "nothing changes" (habit: "none, nothing changes about how they work"; pull: "nothing changes about how I already do this"), that's a signal the candidate is a capability or migration requirement, not a job. Flag it and reconsider whether it belongs in the set, rather than writing the hollow forces later and moving on.

A candidate that fails two or more of these checks should be cut or reframed one level toward the real outcome before it goes in front of the user. Present the surviving list to the user, with the entities each job touches in parentheses. Confirm before writing files. Like model-entities, this is the single most valuable interaction in the skill.

**Skip-confirmation mode.** If the user has explicitly said something like "don't ask for confirmation, just write it" or "stop confirming with me," skip the wait: still show the proposed list above, then move straight to writing instead of pausing for a reply. Unless they scope it narrower ("just for this one," "just for jobs"), treat it as a standing preference for the rest of the session — covering every `model-*` skill and `speck` call from here on, since re-stating it each time would defeat the point. It reverts the moment the user asks to confirm again. Never infer this from a fast or approving reply; it has to be requested explicitly.

### 4. Create one job at a time via `/create-job`

For each confirmed job, delegate the file-writing to the [[create-job]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [job]` + `situation` + `outcome` + `touches` + the four `forces`, prose description, numbered Tasks list), delegates vault prerequisites (`jobs/` folder, `Templates/Job.md`, `.obsidian/templates.json`, graph color group) to [[create-vault]], and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per job, passing:

- Vault path
- Job sentence (sentence-cased verb phrase: `Plan my week`, never `PlanMyWeek`)
- Concrete `situation` and user-POV `outcome`
- Entities touched, grouped by `reads` / `creates` / `updates` / `deletes`
- All four forces (`push`, `pull`, `habit`, `anxiety`) — write `"(none identified — revisit after user interviews)"` for any that genuinely can't be named, rather than dropping the kind
- Prose description (2–4 sentences)
- Numbered Tasks list with the entity each task touches
- Optional success criteria

Recommended order: write the **headline job** first, then habitual jobs, then episodic and emotional ones. This catches naming-convention drift early and makes the rest of the set easier to write.

If the user adds a job mid-flow ("also add one for offboarding"), just call `/create-job` once more. The separation exists for exactly this case.

### 5. Show the user what was created

List the files. Offer next steps:

- Add jobs they think are missing (especially episodic or emotional ones, which tend to get forgotten)
- Drill into any job to flesh out tasks or sharpen the forces
- Cross-check: every entity should be touched by at least one job. Entities no job needs are candidates for deletion.
- Model the performers who execute these jobs with [[model-performers]] — each performer is defined by their main job and explicitly separated from adjacent roles (buyer, approver, reviewer)
- Elaborate any job's thin `## Tasks` sketch into a full step sequence with [[model-workflows]] — and use it for job-less plumbing (login, session refresh) too, which has no home in this skill
- Move on to UI screens or API endpoints — each can cite the job(s) it serves.

## Style notes

- One job per file. Resist the urge to bundle "morning planning" and "evening review" into one note — they're different jobs with different forces.
- Sentence-cased verb phrases for filenames (`Plan my week`, `Capture an idea`) — never PascalCase, never noun-led. The filename should read like the user's own words for the job.
- Link liberally with `[[wiki links]]`. Links to entities that don't exist yet are fine — they appear as unresolved links and signal "we should model this next."
- Don't invent forces the user hasn't implied. If you can't name a credible anxiety, write "(none identified — revisit after user interviews)" rather than fabricating one.
- Don't write implementation, UI mockups, or feature lists. This skill stops at user-outcome modeling.
- JTBD uses "task" for sub-actions inside a job. This is the framework's term — keep it even when the product being modeled has an entity also called `Task`. The two live in different folders (`jobs/` vs `entities/`) and disambiguate by context.
