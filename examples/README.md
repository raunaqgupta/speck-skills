# Examples

Each folder here is a complete Obsidian vault produced by following the Speck skills. Open a folder as a vault in Obsidian (**Open folder as vault**) to browse the notes and see the graph view color-coded by kind.

## `todo-app/`

A personal to-do app where tasks can be captured in the app or by forwarding an email, planned into a daily list, and given reminders.

| Kind | Notes |
| --- | --- |
| Entities | User, List, Task, Reminder |
| Jobs | Get a task out of my head; Decide what to do today; Make sure a deadline doesn't slip |
| Performers | The Capturer, The Planner |
| Surfaces | Todo App, Email Capture |
| Workflows | Sign up; Capture a task; Forward an email as a task; Plan today; Set a reminder |
| Constraints | Capture works without a connection |

A few things it shows:

- **A job reached through two surfaces.** *Get a task out of my head* is delivered by both *Todo App* and *Email Capture*, with one workflow on each.
- **A workflow with no job.** *Sign up* has no `job` key, since signing up isn't something anyone hires the product to do.
- **Implementation kept in a constraint.** Offline support is a constraint bound to *Todo App* and *Capture a task*, so the surface and workflow notes stay implementation-free.
- **Transitions that match entity fields.** Every Transition in the workflow Steps tables names a field or state that exists on the entity's Fields table.
