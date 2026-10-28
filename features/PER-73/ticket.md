---
key: PER-73
type: story
status: todo
blocks: []
blocked_by: []
labels:
- repo:graphql
repos:
- graphql
summary: '`Reminders: in-memory reminder lists and reminders in GraphQL`'
---

Add reminder lists and reminders to the graphql service.

One repo only: graphql. Do not change agent or mcp. No UI, no login, no database: data lives in memory and is empty again when the process restarts.

## Data

ReminderList:

* id: string
* name: string
* reminders: \[Reminder\] — the reminders in this list

Reminder:

* id: string
* listId: string
* text: string
* dueAt: ISO-8601 date-time string
* done: boolean, default false

## Part 1 — Reminder lists

* lists: every list, sorted by name
* list(id: String!): one list, or null if it does not exist
* addList(name: String!): create a list and return it
* renameList(id: String!, name: String!): rename a list and return it
* deleteList(id: String!): delete a list and return true

## Part 2 — Reminders (depends on Part 1)

* reminders(listId: String!, includeDone: Boolean = false): reminders of one list, sorted by dueAt, oldest first
* addReminder(listId: String!, text: String!, dueAt: String!): create a reminder and return it
* completeReminder(id: String!): set done to true and return it

Existing health() and any existing operations stay unchanged.

## Product decisions for review (proposed defaults)

1. Are list names unique? Proposed: yes, case-insensitive; a duplicate is rejected.
2. What happens to reminders when their list is deleted? Proposed: deleted with the list.
3. Is a dueAt in the past allowed? Proposed: rejected with "dueAt must be in the future".
4. Maximum length for list names and reminder text? Proposed: 50 and 200 characters.
5. Are blank values (only spaces) the same as empty? Proposed: yes, rejected.

## Acceptance criteria

* AC1: addList("Home") returns a list with that name and no reminders; lists includes it.
* AC2: addList with an empty, blank, too long or duplicate name is rejected and adds nothing.
* AC3: renameList changes the name; an unknown id or a duplicate name is rejected.
* AC4: deleteList removes the list and its reminders; an unknown id is rejected.
* AC5: addReminder(<Home id>, "Pay rent", <tomorrow>) returns a reminder with that text, dueAt, listId, and done false; list(id).reminders includes it.
* AC6: addReminder with an unknown listId, empty or blank text, text over the limit, an invalid date, or a past date is rejected and adds nothing.
* AC7: reminders returns open reminders of that list sorted by dueAt; done reminders appear only with includeDone: true.
* AC8: completeReminder sets done to true; an unknown id is rejected.
* AC9: health() still returns exactly {"status": "ok"}; existing operations are unchanged.

## Order

Part 1 first, then Part 2 (reminders need lists).
