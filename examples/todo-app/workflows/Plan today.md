---
tags:
  - workflow
surface: "[[Todo App]]"
job: "[[Decide what to do today]]"
performer: "[[The Planner]]"
touches:
  reads:
    - "[[Task]]"
    - "[[List]]"
  updates:
    - "[[Task]]"
---

The Planner builds today's short list from the backlog, deals with yesterday's leftovers, and checks tasks off through the day. It starts on the today view and ends when the day's tasks are done or moved.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | Open the today view; leftovers from earlier days are listed at the top | — | *(not yet designed)* |
| 2 | For each leftover, keep it for today, move it to a later day, or clear its date | [[Task]]: planned_for changed or cleared | *(not yet designed)* |
| 3 | Browse open tasks across [[List]]s, sorted with the nearest due dates first | — | *(not yet designed)* |
| 4 | Add chosen tasks to today | [[Task]]: planned_for → today | *(not yet designed)* |
| 5 | Check off a task when it's done | [[Task]]: open → done (completed_at set) | *(not yet designed)* |
| 6 | Uncheck a task completed by mistake | [[Task]]: done → open (completed_at cleared) | *(not yet designed)* |

## Notes

- Step 5's transition also cancels the Task's scheduled [[Reminder]]s, per the Task invariant.
- Open question: should the today view warn when more tasks are planned than were completed on a typical day?
