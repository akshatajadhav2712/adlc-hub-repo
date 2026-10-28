# Breakdown — PER-73

**Story:** Reminders: in-memory reminder lists and reminders in GraphQL

**Source:** Requirements (PER-73)

---

## Overview

This breakdown splits the feature into two implementation stories following the Part 1 / Part 2 dependency structure defined in the requirements. Story 1 implements reminder lists (queries, mutations, validation), Story 2 implements reminders that belong to those lists. Each story includes its data model, GraphQL schema, resolvers, validation logic, and tests mapped to acceptance criteria.

---

## Story Execution Order

1. **Story 1: Reminder Lists** — must complete before Story 2
2. **Story 2: Reminders** — depends on Story 1

---

## Story 1: Reminder Lists — Create, query, rename, and delete in-memory lists

**Repo:** `graphql`

**Dependencies:** None (opens the epic)

**Covers Requirements:** REQ-1, REQ-2, REQ-3, REQ-4, REQ-5, REQ-6, REQ-7, REQ-8, REQ-14

**Covers Acceptance Criteria:** AC1, AC2, AC3, AC4, AC9

### Description

Implement in-memory reminder lists with the following GraphQL operations:

**Queries:**
- `lists: [ReminderList!]!` — returns all lists sorted by name (case-insensitive)
- `list(id: String!): ReminderList` — returns one list by id, or null if not found

**Mutations:**
- `addList(name: String!): ReminderList!` — creates a new list with a unique id and empty reminders array
- `renameList(id: String!, name: String!): ReminderList!` — updates a list's name
- `deleteList(id: String!): Boolean!` — deletes a list and all its reminders, returns true

**Data Model (ReminderList):**
- `id: String!` (unique identifier, generated)
- `name: String!` (1–50 characters, unique case-insensitive, non-blank)
- `reminders: [Reminder!]!` (empty array for Story 1; Reminder type defined in Story 2)

**Validation Rules:**
1. List names must be unique (case-insensitive comparison)
2. Names must be non-empty, non-blank (no whitespace-only strings), and ≤50 characters
3. `renameList` and `deleteList` reject unknown ids
4. `renameList` rejects duplicate names (case-insensitive)
5. `deleteList` removes the list and cascades to delete all its reminders (no reminders exist in Story 1; Story 2 implements cascade)

**Existing operations:**
- `health()` returns `{"status": "ok"}` unchanged

### Allowed Paths

GraphQL schema, resolvers, in-memory store, and validation logic:
- `src/schema.py` or `src/schema.graphql` (GraphQL type definitions)
- `src/resolvers.py` or `src/resolvers/` (query and mutation resolvers)
- `src/store.py` (in-memory data structure for lists)
- `src/validation.py` (validation helpers for name uniqueness, length, blank checks)

Tests:
- `tests/test_reminder_lists.py`

_(Concrete paths to be confirmed in LLD based on existing `src/health.py` structure)_

### Verification Approach

**Test count: 13 tests** (mapped to requirements and acceptance criteria):

1. **REQ-1 / AC1 (partial):** Query `lists` returns empty array initially; after adding a list, `lists` includes it sorted by name
2. **REQ-2 / AC1 (partial):** Query `list(id)` with valid id returns the list; with unknown id returns null
3. **REQ-3 / AC1:** `addList(name: "Home")` creates a list with that name, a unique id, and empty reminders array
4. **REQ-4 / AC2 (empty name):** `addList(name: "")` rejects with error, adds nothing
5. **REQ-4 / AC2 (blank name):** `addList(name: "   ")` rejects with error, adds nothing
6. **REQ-4 / AC2 (over limit):** `addList(name: "<51-char string>")` rejects with error, adds nothing
7. **REQ-4 / AC2 (duplicate exact):** `addList(name: "Home")` when "Home" exists rejects with error, adds nothing
8. **REQ-4 / AC2 (duplicate case-insensitive):** `addList(name: "home")` when "Home" exists rejects with error, adds nothing
9. **REQ-5 / AC3 (partial):** `renameList(id: "<valid>", name: "Work")` updates the name and returns updated list
10. **REQ-6 / AC3 (unknown id):** `renameList(id: "unknown", name: "Work")` rejects with error, modifies nothing
11. **REQ-6 / AC3 (duplicate name):** `renameList(id: "<valid>", name: "Home")` when another list is named "Home" rejects with error, modifies nothing
12. **REQ-7, REQ-8 / AC4:** `deleteList(id: "<valid>")` removes the list and returns true; `deleteList(id: "unknown")` rejects with error, modifies nothing
13. **REQ-14 / AC9:** `health()` still returns exactly `{"status": "ok"}`; existing operations unchanged

---

## Story 2: Reminders — Create, query, and complete reminders in lists

**Repo:** `graphql`

**Dependencies:** Story 1 (requires ReminderList operations and data model)

**Covers Requirements:** REQ-9, REQ-10, REQ-11, REQ-12, REQ-13, REQ-14

**Covers Acceptance Criteria:** AC5, AC6, AC7, AC8, AC9

### Description

Implement in-memory reminders that belong to reminder lists, with the following GraphQL operations:

**Queries:**
- `reminders(listId: String!, includeDone: Boolean = false): [Reminder!]!` — returns reminders of one list sorted by dueAt (oldest first); filters out done reminders unless `includeDone: true`

**Mutations:**
- `addReminder(listId: String!, text: String!, dueAt: String!): Reminder!` — creates a reminder in the specified list
- `completeReminder(id: String!): Reminder!` — sets done to true and returns the updated reminder

**Data Model (Reminder):**
- `id: String!` (unique identifier, generated)
- `listId: String!` (references a ReminderList; must be valid at creation)
- `text: String!` (1–200 characters, non-blank)
- `dueAt: String!` (ISO-8601 date-time string; must be in the future at creation)
- `done: Boolean!` (default false)

**Validation Rules:**
1. `addReminder` requires a valid `listId` (list must exist)
2. `text` must be non-empty, non-blank, and ≤200 characters
3. `dueAt` must be a valid ISO-8601 date-time string
4. `dueAt` must be in the future at creation time (reject past dates per product decision 3)
5. `completeReminder` rejects unknown reminder ids
6. Cascade: when a list is deleted (via Story 1's `deleteList`), all its reminders are deleted (implement in `deleteList` resolver)

**Existing operations:**
- All Story 1 operations and `health()` remain unchanged

### Allowed Paths

GraphQL schema, resolvers, in-memory store, and validation logic:
- `src/schema.py` or `src/schema.graphql` (add Reminder type and operations)
- `src/resolvers.py` or `src/resolvers/` (add reminder query/mutation resolvers; update `deleteList` for cascade)
- `src/store.py` (add in-memory structure for reminders; link to lists)
- `src/validation.py` (add reminder validation: text length, date parsing, future check)

Tests:
- `tests/test_reminders.py`

_(Concrete paths to be confirmed in LLD based on Story 1 structure)_

### Verification Approach

**Test count: 14 tests** (mapped to requirements and acceptance criteria):

1. **REQ-9 / AC5:** `addReminder(listId: "<valid>", text: "Pay rent", dueAt: "<future ISO-8601>")` creates reminder with unique id, provided text, dueAt, listId, done false; reminder appears in `list(id).reminders`
2. **REQ-10 / AC6 (unknown listId):** `addReminder(listId: "unknown", ...)` rejects with error, adds nothing
3. **REQ-10 / AC6 (empty text):** `addReminder(listId: "<valid>", text: "", ...)` rejects with error, adds nothing
4. **REQ-10 / AC6 (blank text):** `addReminder(listId: "<valid>", text: "   ", ...)` rejects with error, adds nothing
5. **REQ-10 / AC6 (over limit):** `addReminder(listId: "<valid>", text: "<201-char string>", ...)` rejects with error, adds nothing
6. **REQ-10 / AC6 (invalid date):** `addReminder(listId: "<valid>", text: "Task", dueAt: "not-a-date")` rejects with error, adds nothing
7. **REQ-10 / AC6 (past date):** `addReminder(listId: "<valid>", text: "Task", dueAt: "2020-01-01T00:00:00Z")` rejects with error, adds nothing
8. **REQ-11 / AC7 (default excludes done):** `reminders(listId: "<valid>")` returns only open reminders sorted by dueAt oldest first
9. **REQ-11 / AC7 (includeDone true):** `reminders(listId: "<valid>", includeDone: true)` returns all reminders (done and open) sorted by dueAt oldest first
10. **REQ-12 / AC8 (partial):** `completeReminder(id: "<valid>")` sets done to true, returns updated reminder
11. **REQ-13 / AC8 (unknown id):** `completeReminder(id: "unknown")` rejects with error, modifies nothing
12. **REQ-7 (cascade delete):** `deleteList(id: "<valid with reminders>")` deletes the list and all its reminders; subsequent `reminders(listId: "<deleted>")` query finds no reminders (or rejects if listId validation added)
13. **REQ-14 / AC9:** `health()` still returns exactly `{"status": "ok"}`
14. **REQ-14 / AC9:** All Story 1 operations (lists, list, addList, renameList, deleteList) remain unchanged

---

## Acceptance Criteria Coverage

| AC   | Story 1 Tests | Story 2 Tests |
|------|---------------|---------------|
| AC1  | 1, 2, 3       | 1 (partial)   |
| AC2  | 4, 5, 6, 7, 8 | —             |
| AC3  | 9, 10, 11     | —             |
| AC4  | 12            | 12 (cascade)  |
| AC5  | —             | 1             |
| AC6  | —             | 2, 3, 4, 5, 6, 7 |
| AC7  | —             | 8, 9          |
| AC8  | —             | 10, 11        |
| AC9  | 13            | 13, 14        |

**Total test count: 27 tests** (13 in Story 1 + 14 in Story 2)

---

## Notes

- **In-memory storage:** Data lives in module-level or singleton structures (dictionaries, lists); cleared on process restart. No database, no persistence.
- **ID generation:** Use `uuid.uuid4()` or similar to generate unique string ids for lists and reminders.
- **Case-insensitive uniqueness:** Normalize list names to lowercase for comparison; store original case.
- **Date validation:** Parse `dueAt` as ISO-8601; compare to `datetime.now(timezone.utc)` to enforce future constraint.
- **Cascade delete:** Story 2 updates the `deleteList` resolver to remove all reminders with matching `listId` before removing the list.
- **GraphQL framework:** Assume Strawberry, Graphene, or similar; concrete framework and structure confirmed in LLD based on existing `src/health.py`.

---

## Open Questions

None; product decisions (uniqueness, cascade, past dates, limits, blank strings) are resolved per requirements.
