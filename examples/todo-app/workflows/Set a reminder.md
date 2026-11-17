---
tags:
  - workflow
surface: "[[Todo App]]"
job: "[[Make sure a deadline doesn't slip]]"
performer: "[[The Planner]]"
touches:
  reads:
    - "[[Task]]"
  creates:
    - "[[Reminder]]"
  updates:
    - "[[Task]]"
    - "[[Reminder]]"
---

The Planner attaches a [[Reminder]] to a [[Task]] and is prompted about it at the chosen time. It starts on the Task's detail view and ends when the prompt is acted on.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | Open the Task | — | *(not yet designed)* |
| 2 | Set or confirm the due date | [[Task]]: due_date set | *(not yet designed)* |
| 3 | Pick a reminder time, from suggestions relative to the due date or a custom time | [[Reminder]]: none → scheduled | *(not yet designed)* |
| 4 | At the reminder time, receive the prompt | [[Reminder]]: scheduled → sent (sent_at set) | *(not yet designed)* |
| 5 | Open the prompt to land on the Task, then complete it or plan it for today | [[Task]]: open → done, or planned_for → today | *(not yet designed)* |

## Notes

- Removing a reminder before it fires moves it to cancelled rather than deleting it: [[Reminder]]: scheduled → cancelled.
