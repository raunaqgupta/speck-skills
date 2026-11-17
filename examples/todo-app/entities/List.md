---
tags:
  - entity
relations:
  belongs_to:
    - "[[User]]"
  has_many:
    - "[[Task]]"
---

A named grouping of [[Task]]s, such as "Groceries" or "Work". Lists let a [[User]] keep unrelated tasks apart without having to sort them at the moment of capture, since the inbox List catches anything not yet filed.

## Fields

| Field      | Type       | Notes                                        |
| ---------- | ---------- | -------------------------------------------- |
| id         | identifier | Unique identifier                            |
| name       | string     |                                              |
| is_inbox   | boolean    | True for the one List that catches new Tasks |
| position   | number     | Order in the User's list of Lists            |
| created_at | datetime   |                                              |
| updated_at | datetime   |                                              |

## Invariants

- The inbox List cannot be deleted or renamed.
- Deleting any other List deletes its [[Task]]s.
