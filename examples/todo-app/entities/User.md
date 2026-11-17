---
tags:
  - entity
relations:
  has_many:
    - "[[List]]"
  has_many_through:
    - "[[Task]]"   # via [[List]]
---

The person whose tasks the product holds. A User owns every [[List]] and, through them, every [[Task]]; nothing in this model is shared between Users.

## Fields

| Field         | Type       | Notes                                                                 |
| ------------- | ---------- | --------------------------------------------------------------------- |
| id            | identifier | Unique identifier                                                     |
| email         | string     | Unique across Users                                                   |
| display_name  | string     |                                                                       |
| time_zone     | string     | Used to decide what "today" means and when a [[Reminder]] is due      |
| inbox_address | string     | Personal forwarding address; mail sent here becomes a [[Task]]        |
| created_at    | datetime   |                                                                       |
| updated_at    | datetime   |                                                                       |

## Relationship notes

- One of the User's [[List]]s is the inbox: the List with `is_inbox` set. It is created along with the User and is where a captured [[Task]] lands when no List is chosen.

## Invariants

- Every User has exactly one inbox [[List]].
- `inbox_address` is unique across Users and never reused after a User is gone.
