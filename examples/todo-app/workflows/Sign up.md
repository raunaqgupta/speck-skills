---
tags:
  - workflow
surface: "[[Todo App]]"
touches:
  creates:
    - "[[User]]"
    - "[[List]]"
---

A new person creates an account and lands on an empty inbox ready for their first [[Task]]. It has no job of its own; it is the entry point every job depends on.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | Enter email and display name | — | *(not yet designed)* |
| 2 | Confirm the email address | [[User]]: none → created | *(not yet designed)* |
| 3 | Confirm the detected time zone, or change it | [[User]]: time_zone set | *(not yet designed)* |
| 4 | Land on the empty inbox, with the personal forwarding address shown | [[List]]: none → created (is_inbox) | *(not yet designed)* |
