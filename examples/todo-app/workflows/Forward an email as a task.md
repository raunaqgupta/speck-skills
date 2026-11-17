---
tags:
  - workflow
surface: "[[Email Capture]]"
job: "[[Get a task out of my head]]"
performer: "[[The Capturer]]"
touches:
  reads:
    - "[[User]]"
  creates:
    - "[[Task]]"
---

The Capturer forwards or writes an email to their personal inbox address, and it becomes a [[Task]] in their inbox [[List]]. It starts in their mail client and ends with a confirmation reply.

## Steps

| # | Action | Transition | Screen |
| - | ------ | ---------- | ------ |
| 1 | Forward a message, or write a new one, to the personal inbox address | — | *(not yet designed)* |
| 2 | The address is matched to its [[User]] | — | — |
| 3 | The subject becomes the title and the body becomes the notes | [[Task]]: none → open (source: email) | — |
| 4 | Receive a short confirmation reply linking to the new Task | — | *(not yet designed)* |

## Notes

- Mail to an address that matches no User is dropped without a reply, so the address can't be probed.
- Open question: should a list name in the subject (e.g. "#groceries") file the Task directly instead of using the inbox?
