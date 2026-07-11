---
name: model-performers
description: Model the job performers of a product — the functional roles defined by the job they execute, not by who they are — by creating one Markdown note per performer inside an Obsidian vault's `performers/` folder. A performer is whoever carries out the main job: not the buyer, not the approver, not the manager, but the executor. Use this skill when the user is planning a product and the conversation turns to "who does this job?", "who is the executor?", "what role performs this?", or "let's define the performers." Also trigger when the user says "personas" but means functional actors — redirect them to this framework. Performers are defined by the functional act, not by demographics: the same individual may be a different performer in a different job context, and the definition stays stable as specific individuals come and go. Pairs with model-jobs (the jobs performers execute) and model-entities (the objects the jobs touch).
---

# Performer Modeling

This skill turns product-planning conversations into a navigable set of job performer notes in an Obsidian vault. Each note defines one functional role: the executor of a main job, separated from adjacent roles in the ecosystem, and described purely in terms of the act being performed. Performers link to job notes via `[[wiki links]]`, closing the loop between *what* the product is (entities), *why* people use it (jobs), and *who executes those jobs* (performers).

## Why this exists

Traditional user definitions (personas, segments, archetypes) drift toward demographics and psychographics — descriptors that feel concrete but don't drive design decisions. The performer definition starts from the opposite end: the functional act. Who is actually carrying out this job? What is their role in getting it done? By anchoring the definition to the job, the model stays honest (you can't fabricate a performer without a job to attach them to) and stable (as individuals change, the functional role persists).

The performer framework also forces a discipline that personas skip: separating the executor from adjacent roles. The person who buys a task manager (the buyer), the manager who approves which tool the team uses (the approver), and the person who actually uses it daily to plan their week (the performer) have entirely different needs. Conflating them is how products end up serving the person who writes the check rather than the person who does the job.

## When to use

Trigger this skill when the user is **planning or validating** a product and the conversation turns to who performs the jobs. Examples:

- "Who actually does this job?"
- "Let's define our performers / actors / users."
- "Who is the executor of this?"
- "What roles do we need to design for?"
- Jobs have been modeled and the user now wants to identify who executes them.

Redirect gently when the user says "personas" but means functional roles — explain the distinction and proceed with performer modeling.

Do **not** trigger for buyer analysis, market segmentation, or org chart mapping. Do **not** trigger when the user wants jobs (use [[model-jobs]]) or entities (use [[model-entities]]). Do **not** conflate the performer with adjacent roles like buyers, approvers, or reviewers — define those separately only if they also have jobs in the vault.

## Workflow

### 1. Identify the vault

Same as the [[model-entities]] skill. If a vault already exists for this product (likely, since performers usually come after job and entity work), use it. Otherwise see `model-entities`'s step 1 for the vault-discovery flow.

### 2. Check for existing jobs

Before proposing performers, check whether the vault has a `jobs/` folder with job notes. Performers without jobs have nothing to anchor to — if jobs don't exist, offer to run [[model-jobs]] first. Proceed without them only if the user wants to sketch performers speculatively; flag which jobs each performer implies as unresolved links.

### 3. Propose the performer set before writing

Read the existing job notes and draft one performer per distinct functional role. A few rules:

- **One performer per main job** — if two jobs have genuinely different executors, they need separate performers. If the same functional role executes both, one performer with `also_performs` covers it.
- **Default to fewer** — most products have 2–4 performers. If you're drafting more than five, check whether some are really the same functional role in slightly different contexts.
- **Name them by act, not by person** — `The Planner`, `The Capturer`, `The Collaborator`. Never personal names, never demographic descriptors.
- **Identify who is NOT the performer** — for each candidate, name the adjacent roles they should be distinguished from (buyer, approver, reviewer, manager, audience, assistant). This is the single most clarifying exercise in performer modeling.

For each candidate, present:
- Functional role name
- The main job they execute (wiki link)
- Any secondary jobs they also perform
- The two or three adjacent roles they need to be distinguished from

Present the list to the user and confirm before writing files. This proposal is the most valuable interaction in the skill — it surfaces conflation early.

### 4. Create one performer at a time via `/create-performer`

For each confirmed performer, delegate the file-writing to the [[create-performer]] skill. That skill owns the canonical artifact shape (frontmatter `tags: [performer]` + `main_job` + optional `also_performs`, functional prose definition, Distinct From section, Context of Execution section), handles vault prerequisites (`performers/` folder, `Templates/Performer.md`, `.obsidian/templates.json`), and refuses to overwrite existing files. **Do not rewrite that template inline here.**

Call it once per performer, passing:

- Vault path
- Performer name (functional role label, title-case noun phrase)
- Main job (`[[wiki link]]` to the single defining job)
- Also performs — secondary jobs, if any
- Functional definition (2–3 sentences describing the act, not the person)
- Distinct from (2–4 adjacent roles with one-line explanations)
- Context of execution (the triggering condition, framed as a situation not a person)

Recommended order: write the performer for the **headline job** first, then secondary performers. This catches definitional overlap early — if two performers seem to need the same "distinct from" exclusions, they may be the same performer.

### 5. Show the user what was created

List the files. Offer next steps:

- Add performers for any jobs that have no executor defined
- Sharpen any "distinct from" sections where the boundary is still fuzzy
- Cross-check: every job should have at least one performer as its executor. Jobs with no performer are either support jobs (worth noting) or candidates for removal.
- Model the concrete interaction sequences each performer executes with [[model-workflows]] — each can cite the performer it serves and the job it supports.

## Style notes

- **One performer per file.** If you're tempted to combine two roles, ask whether they ever have conflicting needs from the product. If yes, they must be separate.
- **Functional role names.** `The Planner`, `The Reviewer`, `The Coordinator`. Not personal names, not job titles, not adjectives. The name should describe the act, not the actor.
- **The functional definition test.** Read the prose back. If you could substitute any individual filling that role and have the definition still apply, it's functional. If it depends on personal traits, reframe.
- **Distinct from is not an exhaustive org chart.** Only list the adjacent roles that a reader would actually confuse with this performer. Three sharp bullets beat ten obvious ones.
- **Context is a situation, not a lifestyle.** "When the backlog is unbounded and needs bounding" not "on Sunday evenings when feeling overwhelmed." The former holds across anyone in that situation; the latter is demographic.
- **Don't invent performers the jobs don't imply.** If no job in the vault requires a buyer or an admin, don't define them here.
