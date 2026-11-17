---
tags:
  - entity
relations:
  belongs_to:
    - "[[Task]]"
---

A prompt sent to the [[User]] at a chosen time about one [[Task]]. Reminders exist for tasks where the time matters more than the list they sit in.

## Fields

| Field      | Type                                | Notes                              |
| ---------- | ----------------------------------- | ---------------------------------- |
| id         | identifier                          | Unique identifier                  |
| remind_at  | datetime                            | In the User's time zone            |
| status     | one of: scheduled, sent, cancelled  |                                    |
| sent_at    | datetime                            | Set when `status` becomes sent     |
| created_at | datetime                            |                                    |
| updated_at | datetime                            |                                    |

## Invariants

- A Reminder can only be scheduled for a time in the future.
- A Reminder on a done [[Task]] is never sent.
