---
tags:
  - workflow
surface: "[[Todo App]]"
job: "[[Get a task out of my head]]"
performer: "[[The Capturer]]"
touches:
  reads:
    - "[[List]]"
  creates:
    - "[[Task]]"
---

The Capturer adds a [[Task]] from anywhere in the app with as little input as possible. It starts from a single capture affordance and ends with the Task saved and the person back where they were.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | Open quick capture from any screen | — | *(not yet designed)* |
| 2 | Type a title | — | *(not yet designed)* |
| 3 | Optionally pick a [[List]] (defaults to the inbox) and a due date | — | *(not yet designed)* |
| 4 | Save | [[Task]]: none → open (source: app) | *(not yet designed)* |
| 5 | Capture closes; the previous screen is restored | — | *(not yet designed)* |

## Notes

- Step 3 is skippable by design; saving straight from step 2 must always work.
