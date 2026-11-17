---
tags:
  - entity
relations:
  belongs_to:
    - "[[List]]"
  has_many:
    - "[[Reminder]]"
---

A single thing the [[User]] intends to do. A Task sits in one [[List]], can be planned for a specific day, and can carry [[Reminder]]s so it isn't forgotten.

## Fields

| Field        | Type                 | Notes                                                    |
| ------------ | -------------------- | -------------------------------------------------------- |
| id           | identifier           | Unique identifier                                        |
| title        | string               | Required; the only field needed at capture               |
| notes        | string               | Optional longer text                                     |
| status       | one of: open, done   |                                                          |
| due_date     | date                 | Optional; the date it must be done by                    |
| planned_for  | date                 | Optional; the day the User intends to work on it         |
| source       | one of: app, email   | How the Task was captured                                |
| completed_at | datetime             | Set when `status` becomes done                           |
| created_at   | datetime             |                                                          |
| updated_at   | datetime             |                                                          |

## Relationship notes

- `due_date` and `planned_for` are separate on purpose: a Task due Friday can be planned for Wednesday. "Today" in the product means `planned_for` equals the current date in the [[User]]'s time zone.

## Invariants

- `completed_at` is set if and only if `status` is done.
- Completing a Task cancels its scheduled [[Reminder]]s.
