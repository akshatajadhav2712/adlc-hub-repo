# Requirements — PER-73

**Story:** Reminders: in-memory reminder lists and reminders in GraphQL

**Source:** Ticket PER-73

---

## Overview

This feature adds in-memory reminder list and reminder management to the GraphQL service. Data persists only while the process runs; a restart clears all lists and reminders. The work splits into two parts: Part 1 adds reminder lists, Part 2 adds reminders that belong to those lists.

---

## Product Decisions

The ticket proposes five product decisions that require review before implementation:

1. **List name uniqueness:** Are list names unique (case-insensitive)? **Proposed: yes**
2. **Cascading deletes:** What happens to reminders when their list is deleted? **Proposed: deleted with the list**
3. **Past dates:** Is a dueAt in the past allowed? **Proposed: rejected with "dueAt must be in the future"**
4. **Length limits:** Maximum length for list names and reminder text? **Proposed: 50 and 200 characters**
5. **Blank values:** Are blank values (only spaces) the same as empty? **Proposed: yes, rejected**

These decisions are incorporated into the requirements below as proposed; TPM review may adjust them.

---

## Requirements

### Part 1: Reminder Lists

#### REQ-1: Query all reminder lists

**Given** the service is running  
**When** the user queries `lists`  
**Then** the service returns all reminder lists sorted by name (case-insensitive)  
**And** each list includes its id, name, and reminders array

**Acceptance Criterion:** AC1 (partial — lists includes the new list after addList)

---

#### REQ-2: Query a single reminder list by ID

**Given** the service is running  
**When** the user queries `list(id: "<valid-id>")`  
**Then** the service returns the list with that id  

**Given** the service is running  
**When** the user queries `list(id: "<unknown-id>")`  
**Then** the service returns null

**Acceptance Criterion:** AC1 (partial — list query used to verify creation)

---

#### REQ-3: Create a reminder list

**Given** the service is running  
**And** no list exists with the name "Home" (case-insensitive)  
**When** the user calls `addList(name: "Home")`  
**Then** the service creates a new list with a unique id, name "Home", and an empty reminders array  
**And** the service returns the created list  

**Acceptance Criterion:** AC1

---

#### REQ-4: Reject invalid list creation

**Given** the service is running  
**When** the user calls `addList` with a name that is empty, blank (only spaces), exceeds 50 characters, or matches an existing list name (case-insensitive)  
**Then** the service rejects the operation with an appropriate error  
**And** the service does not create a list

**Edge cases:**
- Empty string: `addList(name: "")`
- Blank string: `addList(name: "   ")`
- Over limit: `addList(name: "<51-character-string>")`
- Duplicate (exact match): `addList(name: "Home")` when "Home" exists
- Duplicate (case-insensitive): `addList(name: "home")` when "Home" exists

**Acceptance Criterion:** AC2

---

#### REQ-5: Rename a reminder list

**Given** a list exists with id "abc123" and name "Home"  
**And** no other list has the name "Work" (case-insensitive)  
**When** the user calls `renameList(id: "abc123", name: "Work")`  
**Then** the service updates the list's name to "Work"  
**And** the service returns the updated list

**Acceptance Criterion:** AC3 (partial — successful rename)

---

#### REQ-6: Reject invalid list rename

**Given** the service is running  
**When** the user calls `renameList` with an unknown id or a name that duplicates an existing list (case-insensitive)  
**Then** the service rejects the operation with an appropriate error  
**And** the service does not modify any list

**Edge cases:**
- Unknown id: `renameList(id: "unknown", name: "Work")`
- Duplicate name: `renameList(id: "abc123", name: "Home")` when another list is named "Home"

**Acceptance Criterion:** AC3 (partial — rejection cases)

---

#### REQ-7: Delete a reminder list and its reminders

**Given** a list exists with id "abc123"  
**And** the list contains zero or more reminders  
**When** the user calls `deleteList(id: "abc123")`  
**Then** the service deletes the list  
**And** the service deletes all reminders belonging to that list  
**And** the service returns true

**Acceptance Criterion:** AC4 (partial — successful delete, cascading delete per product decision 2)

---

#### REQ-8: Reject delete of unknown list

**Given** the service is running  
**When** the user calls `deleteList(id: "<unknown-id>")`  
**Then** the service rejects the operation with an appropriate error  
**And** the service does not modify any data

**Acceptance Criterion:** AC4 (partial — rejection case)

---

### Part 2: Reminders (depends on Part 1)

#### REQ-9: Create a reminder in a list

**Given** a list exists with id "abc123"  
**When** the user calls `addReminder(listId: "abc123", text: "Pay rent", dueAt: "<ISO-8601 future date>")`  
**Then** the service creates a new reminder with a unique id, the provided text and dueAt, listId "abc123", and done false  
**And** the service returns the created reminder  
**And** the reminder appears in the list's reminders array

**Acceptance Criterion:** AC5

---

#### REQ-10: Reject invalid reminder creation

**Given** the service is running  
**When** the user calls `addReminder` with an unknown listId, empty or blank text, text exceeding 200 characters, an invalid ISO-8601 date, or a past date  
**Then** the service rejects the operation with an appropriate error  
**And** the service does not create a reminder

**Edge cases:**
- Unknown listId: `addReminder(listId: "unknown", text: "Task", dueAt: "<future>")`
- Empty text: `addReminder(listId: "abc123", text: "", dueAt: "<future>")`
- Blank text: `addReminder(listId: "abc123", text: "   ", dueAt: "<future>")`
- Over limit: `addReminder(listId: "abc123", text: "<201-character-string>", dueAt: "<future>")`
- Invalid date: `addReminder(listId: "abc123", text: "Task", dueAt: "not-a-date")`
- Past date: `addReminder(listId: "abc123", text: "Task", dueAt: "2020-01-01T00:00:00Z")` (per product decision 3)

**Acceptance Criterion:** AC6

---

#### REQ-11: Query reminders of a list

**Given** a list exists with id "abc123"  
**And** the list contains reminders with various done states and dueAt values  
**When** the user calls `reminders(listId: "abc123", includeDone: false)`  
**Then** the service returns only reminders where done is false, sorted by dueAt (oldest first)

**Given** a list exists with id "abc123"  
**And** the list contains reminders with various done states and dueAt values  
**When** the user calls `reminders(listId: "abc123", includeDone: true)`  
**Then** the service returns all reminders (done and not done), sorted by dueAt (oldest first)

**Acceptance Criterion:** AC7

---

#### REQ-12: Complete a reminder

**Given** a reminder exists with id "xyz789" and done is false  
**When** the user calls `completeReminder(id: "xyz789")`  
**Then** the service sets done to true  
**And** the service returns the updated reminder

**Acceptance Criterion:** AC8 (partial — successful completion)

---

#### REQ-13: Reject completion of unknown reminder

**Given** the service is running  
**When** the user calls `completeReminder(id: "<unknown-id>")`  
**Then** the service rejects the operation with an appropriate error  
**And** the service does not modify any data

**Acceptance Criterion:** AC8 (partial — rejection case)

---

#### REQ-14: Existing operations remain unchanged

**Given** the service is running  
**When** the user calls `health()`  
**Then** the service returns exactly `{"status": "ok"}`  
**And** any other existing operations continue to function as before

**Acceptance Criterion:** AC9

---

## Data Model (Reference)

**ReminderList:**
- id: string (unique identifier)
- name: string (1–50 characters, unique case-insensitive, non-blank)
- reminders: array of Reminder objects

**Reminder:**
- id: string (unique identifier)
- listId: string (references a ReminderList)
- text: string (1–200 characters, non-blank)
- dueAt: ISO-8601 date-time string (must be in the future at creation)
- done: boolean (default false)

---

## Coverage Map

| Requirement | Acceptance Criteria |
|-------------|---------------------|
| REQ-1       | AC1 (partial)       |
| REQ-2       | AC1 (partial)       |
| REQ-3       | AC1                 |
| REQ-4       | AC2                 |
| REQ-5       | AC3 (partial)       |
| REQ-6       | AC3 (partial)       |
| REQ-7       | AC4 (partial)       |
| REQ-8       | AC4 (partial)       |
| REQ-9       | AC5                 |
| REQ-10      | AC6                 |
| REQ-11      | AC7                 |
| REQ-12      | AC8 (partial)       |
| REQ-13      | AC8 (partial)       |
| REQ-14      | AC9                 |

---

## Ordering Constraint

Part 1 (REQ-1 through REQ-8) must be completed before Part 2 (REQ-9 through REQ-13) because reminders depend on the existence of lists. REQ-14 applies to both parts.
