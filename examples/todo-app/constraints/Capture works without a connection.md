---
tags:
  - constraint
applies_to:
  - "[[Todo App]]"
  - "[[Capture a task]]"
---

Capturing a [[Task]] in the [[Todo App]] must succeed with no network connection. Tasks captured offline are saved on the device and reach the person's other devices once a connection returns, without the person having to do anything.

## Rationale

[[Get a task out of my head]] fails if capture ever stalls, and the moments a to-do comes to mind (commuting, travelling, in a basement meeting room) are often the ones with poor connectivity.

## Implications

- The app keeps a local copy of the person's Lists so a captured Task can be filed without a round trip.
- Two devices editing the same Task offline need a defined way to reconcile; capture itself never conflicts because it only creates.
