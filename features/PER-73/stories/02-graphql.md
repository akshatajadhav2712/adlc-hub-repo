# LLD — Story 02-graphql: Reminders

**Repo:** `graphql`

**Covers:** AC5, AC6, AC7, AC8, AC9

**Depends on:** Story 01-graphql (Reminder Lists)

---

## Acceptance Criteria (from requirements + breakdown)

- **AC5:** addReminder(<Home id>, "Pay rent", <tomorrow>) returns a reminder with that text, dueAt, listId, description (if provided), and done false; list(id).reminders includes it.
- **AC6:** addReminder with an unknown listId, empty or blank text, text over the limit, an invalid date, or a past date is rejected and adds nothing.
- **AC7:** reminders returns open reminders of that list sorted by dueAt; done reminders appear only with includeDone: true.
- **AC8:** completeReminder sets done to true; an unknown id is rejected.
- **AC9:** health() still returns exactly {"status": "ok"}; existing operations are unchanged.

---

## File Plan

All paths below exist in the repo tree or are new files in established directories.

### Modified files

- `src/schema.py` — Add reminder queries and mutations to existing schema; add description field to Reminder type
- `src/resolvers.py` — Add reminder resolver functions; update `deleteList` for cascade
- `src/store.py` — Add in-memory structure for reminders; update `delete_list` for cascade
- `src/validation.py` — Add reminder validation (text length, description length, date parsing, future check)

### New files

- `tests/test_reminders.py` — Tests for this story's operations

### Unchanged files

- `README.md`
- `src/__init__.py`
- `src/health.py`
- `tests/__init__.py`
- `tests/test_health.py`
- `tests/test_reminder_lists.py` — Story 1 tests remain unchanged

---

## Data Model

### Reminder

Represented as a Python dictionary (formalized as GraphQL type in `src/schema.py`):

```python
{
    "id": str,             # Unique identifier (UUID v4)
    "listId": str,         # References a ReminderList; must exist at creation
    "text": str,           # 1–200 characters, non-blank
    "description": str | None,  # Optional, 0–500 characters if provided
    "dueAt": str,          # ISO-8601 date-time string; must be future at creation
    "done": bool           # Default False
}
```

**Invariants:**
- `id` is unique across all reminders
- `listId` references a valid ReminderList (validated at creation)
- `text` is non-empty, non-blank (no whitespace-only), ≤200 characters
- `description` is optional; if provided, ≤500 characters, can be empty/blank (stored as None)
- `dueAt` is a valid ISO-8601 string and represents a future datetime at creation
- `done` defaults to `False`

---

## Interfaces

### GraphQL Schema (`src/schema.py`)

**Additions to existing schema:**

```graphql
# Reminder type updated from Story 1 to include description field
type Reminder {
  id: String!
  listId: String!
  text: String!
  description: String
  dueAt: String!
  done: Boolean!
}

# Add to Query type
extend type Query {
  reminders(listId: String!, includeDone: Boolean = false): [Reminder!]!
}

# Add to Mutation type
extend type Mutation {
  addReminder(listId: String!, text: String!, description: String, dueAt: String!): Reminder!
  completeReminder(id: String!): Reminder!
}
```

**Notes:**
- `description` field added to `Reminder` type (nullable String)
- `addReminder` mutation accepts optional `description` parameter
- `ReminderList.reminders` field now populated with actual reminder data including description

---

### Resolver Signatures (`src/resolvers.py`)

**New resolvers:**

```python
# Queries
def resolve_reminders(info, listId: str, includeDone: bool = False) -> list[dict]:
    """Returns reminders of one list sorted by dueAt (oldest first).
    Filters out done reminders unless includeDone is True."""
    ...

# Mutations
def resolve_add_reminder(info, listId: str, text: str, dueAt: str, description: str | None = None) -> dict:
    """Creates a reminder in the specified list.
    Raises GraphQLError on validation failure (unknown listId, invalid text/description/date, past date)."""
    ...

def resolve_complete_reminder(info, id: str) -> dict:
    """Sets done to True and returns updated reminder.
    Raises GraphQLError if id unknown."""
    ...
```

**Updated resolver:**

```python
def resolve_delete_list(info, id: str) -> bool:
    """Deletes a list and all its reminders.
    Raises GraphQLError if id unknown.
    (Updated from Story 1 to cascade-delete reminders)"""
    ...
```

**Error Handling:**
- Validation failures and unknown ids raise `graphql.GraphQLError` with descriptive messages.
- Errors propagate to GraphQL response as structured error objects.

---

### In-Memory Store (`src/store.py`)

**Additions to existing store:**

```python
from datetime import datetime, timezone
from typing import Dict, List
from uuid import uuid4

# Existing from Story 1:
# _lists: Dict[str, dict] — keyed by list id
# _name_index: Dict[str, str] — lowercase name -> list id

# New storage for reminders
_reminders: Dict[str, dict] = {}  # keyed by reminder id
_list_reminders: Dict[str, List[str]] = {}  # list_id -> [reminder_ids]

def get_reminders_for_list(list_id: str, include_done: bool = False) -> List[dict]:
    """Returns reminders for a list sorted by dueAt (oldest first).
    Filters out done reminders unless include_done is True."""
    ...

def get_reminder_by_id(reminder_id: str) -> dict | None:
    """Returns a reminder by id, or None if not found."""
    ...

def add_reminder(list_id: str, text: str, due_at: str, description: str | None = None) -> dict:
    """Creates a new reminder with unique id and done=False.
    Raises ValueError on validation failure (unknown listId, invalid text/description/date, past date)."""
    ...

def complete_reminder(reminder_id: str) -> dict:
    """Sets done to True; raises ValueError if id unknown."""
    ...

def delete_list(list_id: str) -> bool:
    """Deletes a list and all its reminders (cascade).
    Raises ValueError if id unknown.
    (Updated from Story 1 to delete associated reminders)"""
    ...
```

**Implementation Notes:**
- `_reminders` maps `id -> reminder_dict` for O(1) lookup.
- `_list_reminders` maps `list_id -> [reminder_ids]` for efficient cascade delete and filtered queries.
- `add_reminder` validates `list_id` exists via `get_list_by_id(list_id)`.
- `description` field stored as None if not provided or if empty/blank string passed.
- `delete_list` updated to iterate `_list_reminders[list_id]` and remove each reminder from `_reminders`, then clear the list entry.
- Sorting by `dueAt` requires parsing ISO-8601 strings for comparison.

---

### Validation Helpers (`src/validation.py`)

**New validation functions:**

```python
from datetime import datetime, timezone

def validate_reminder_text(text: str) -> None:
    """Raises ValueError if text is empty, blank, or exceeds 200 characters."""
    ...

def validate_reminder_description(description: str | None) -> str | None:
    """Validates description field. Returns None if input is None, empty, or blank.
    Raises ValueError if exceeds 500 characters."""
    ...

def parse_iso8601(date_str: str) -> datetime:
    """Parses ISO-8601 date-time string; raises ValueError if invalid."""
    ...

def validate_future_date(date_str: str) -> None:
    """Raises ValueError if parsed datetime is not in the future (UTC comparison)."""
    ...
```

**Rules:**
- **text:** Empty string (`""`) → ValueError; whitespace-only (`"   "`) → ValueError; length > 200 → ValueError
- **description:** None, empty, or whitespace-only → normalized to None (valid); length > 500 → ValueError
- **dueAt:** Invalid ISO-8601 format → ValueError; past datetime (≤ now UTC) → ValueError ("dueAt must be in the future")

---

## Algorithm Notes

### addReminder(listId: String!, text: String!, dueAt: String!, description: String?)

1. Validate list exists: `list_dict = get_list_by_id(listId)`; if None, raise ValueError("List not found")
2. Validate text: `validate_reminder_text(text)` (raises if empty, blank, or >200 chars)
3. Validate description: `normalized_description = validate_reminder_description(description)` (returns None if empty/blank, raises if >500 chars)
4. Validate date format: `parsed_date = parse_iso8601(dueAt)` (raises if invalid)
5. Validate future: `validate_future_date(dueAt)` (raises if past or present)
6. Generate id: `reminder_id = str(uuid4())`
7. Create reminder dict: `{"id": reminder_id, "listId": listId, "text": text, "description": normalized_description, "dueAt": dueAt, "done": False}`
8. Store reminder: `_reminders[reminder_id] = reminder_dict`
9. Link to list: `_list_reminders.setdefault(listId, []).append(reminder_id)`
10. Return reminder dict

### reminders(listId: String!, includeDone: Boolean = false)

1. Get reminder ids: `reminder_ids = _list_reminders.get(listId, [])`
2. Fetch reminders: `reminders_list = [_reminders[rid] for rid in reminder_ids]`
3. Filter by done: if not `includeDone`, filter out reminders where `done == True`
4. Sort by dueAt: `sorted(reminders_list, key=lambda r: parse_iso8601(r["dueAt"]))`
5. Return sorted list

### completeReminder(id: String!)

1. Get reminder: `reminder_dict = get_reminder_by_id(id)`; if None, raise ValueError("Reminder not found")
2. Update done: `reminder_dict["done"] = True`
3. Return updated reminder dict

### deleteList(id: String!) — Updated from Story 1

1. Get list: `list_dict = get_list_by_id(id)`; if None, raise ValueError("List not found")
2. Get reminder ids: `reminder_ids = _list_reminders.get(id, [])`
3. Delete each reminder: for each `rid` in `reminder_ids`, `del _reminders[rid]`
4. Clear list reminders: `del _list_reminders[id]` (if exists)
5. Remove from name index: `normalized = _normalize_name(list_dict["name"])`; `del _name_index[normalized]`
6. Delete list: `del _lists[id]`
7. Return True

### ReminderList.reminders field resolver (implicit)

When querying `list(id).reminders` or `lists.reminders`:
1. For each list, resolve `reminders` field by calling `get_reminders_for_list(list_id, include_done=True)` (include all reminders for this field; filtering by done is done via the `reminders` query)
2. Return the list of reminder dicts

**Note:** The `ReminderList.reminders` field returns all reminders (done and open) for consistency; the `reminders(listId, includeDone)` query provides filtered access.

---

## Tests

**File:** `tests/test_reminders.py`

**Test count: 14 tests** (maps 1:1 to breakdown verification approach; complete AC coverage)

### Test cases (mapped to requirements and breakdown)

1. **REQ-9 / AC5 / Breakdown #1:** `test_add_reminder_success` — `addReminder(listId: "<valid>", text: "Pay rent", dueAt: "<future ISO-8601>")` creates reminder with unique id, provided text, dueAt, listId, description None, done false; `list(id).reminders` includes it
2. **REQ-10 / AC6 (unknown listId) / Breakdown #2:** `test_add_reminder_unknown_list` — `addReminder(listId: "unknown", ...)` raises GraphQLError, adds nothing
3. **REQ-10 / AC6 (empty text) / Breakdown #3:** `test_add_reminder_empty_text` — `addReminder(listId: "<valid>", text: "", ...)` raises GraphQLError, adds nothing
4. **REQ-10 / AC6 (blank text) / Breakdown #4:** `test_add_reminder_blank_text` — `addReminder(listId: "<valid>", text: "   ", ...)` raises GraphQLError, adds nothing
5. **REQ-10 / AC6 (over limit) / Breakdown #5:** `test_add_reminder_text_too_long` — `addReminder(listId: "<valid>", text: "<201-char string>", ...)` raises GraphQLError, adds nothing
6. **REQ-10 / AC6 (invalid date) / Breakdown #6:** `test_add_reminder_invalid_date` — `addReminder(listId: "<valid>", text: "Task", dueAt: "not-a-date")` raises GraphQLError, adds nothing
7. **REQ-10 / AC6 (past date) / Breakdown #7:** `test_add_reminder_past_date` — `addReminder(listId: "<valid>", text: "Task", dueAt: "2020-01-01T00:00:00Z")` raises GraphQLError, adds nothing
8. **REQ-11 / AC7 (default excludes done) / Breakdown #8:** `test_reminders_excludes_done_by_default` — `reminders(listId: "<valid>")` returns only open reminders sorted by dueAt oldest first
9. **REQ-11 / AC7 (includeDone true) / Breakdown #9:** `test_reminders_includes_done_when_requested` — `reminders(listId: "<valid>", includeDone: true)` returns all reminders (done and open) sorted by dueAt oldest first
10. **REQ-12 / AC8 (success) / Breakdown #10:** `test_complete_reminder_success` — `completeReminder(id: "<valid>")` sets done to true, returns updated reminder
11. **REQ-13 / AC8 (unknown id) / Breakdown #11:** `test_complete_reminder_unknown_id` — `completeReminder(id: "unknown")` raises GraphQLError, modifies nothing
12. **REQ-7 (cascade delete) / Breakdown #12:** `test_delete_list_cascades_reminders` — `deleteList(id: "<valid with reminders>")` deletes list and all its reminders; subsequent `reminders(listId: "<deleted>")` returns empty array
13. **REQ-14 / AC9 / Breakdown #13:** `test_health_unchanged` — `health()` returns exactly `{"status": "ok"}`
14. **REQ-14 / AC9 / Breakdown #14:** `test_story1_operations_unchanged` — All Story 1 operations (lists, list, addList, renameList, deleteList structure) remain unchanged; cascade delete is transparent extension

### Additional tests for description field (review feedback)

15. **REQ-9 / AC5 (description field) / Review feedback:** `test_add_reminder_with_description` — `addReminder(listId: "<valid>", text: "Pay rent", description: "Monthly rent payment", dueAt: "<future>")` creates reminder with description field populated
16. **REQ-10 / AC6 (description over limit) / Review feedback:** `test_add_reminder_description_too_long` — `addReminder(listId: "<valid>", text: "Task", description: "<501-char string>", dueAt: "<future>")` raises GraphQLError, adds nothing

**Total test count: 16 tests** (14 from breakdown + 2 for description field per review feedback)

### Test structure

Each test:
1. Sets up initial state (create lists, reminders if needed)
2. Executes GraphQL query or mutation
3. Asserts expected result or error
4. Verifies no side effects (e.g. reminder count unchanged on rejection)

**Example (pseudo-code):**

```python
from datetime import datetime, timedelta, timezone

def test_add_reminder_success():
    # Setup: create a list
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    # Create reminder with future date, no description
    future_date = (datetime.now(timezone.utc) + timedelta(days=1)).isoformat()
    result = execute_graphql(f'''
        mutation {{
            addReminder(listId: "{list_id}", text: "Pay rent", dueAt: "{future_date}") {{
                id listId text description dueAt done
            }}
        }}
    ''')
    
    reminder = result["data"]["addReminder"]
    assert reminder["text"] == "Pay rent"
    assert reminder["description"] is None
    assert reminder["dueAt"] == future_date
    assert reminder["listId"] == list_id
    assert reminder["done"] is False
    
    # Verify it appears in list.reminders
    list_query = execute_graphql(f'query {{ list(id: "{list_id}") {{ reminders {{ id }} }} }}')
    assert len(list_query["data"]["list"]["reminders"]) == 1

def test_add_reminder_with_description():
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    future_date = (datetime.now(timezone.utc) + timedelta(days=1)).isoformat()
    result = execute_graphql(f'''
        mutation {{
            addReminder(
                listId: "{list_id}", 
                text: "Pay rent", 
                description: "Monthly rent payment",
                dueAt: "{future_date}"
            ) {{
                id description
            }}
        }}
    ''')
    
    reminder = result["data"]["addReminder"]
    assert reminder["description"] == "Monthly rent payment"

def test_add_reminder_description_too_long():
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    future_date = (datetime.now(timezone.utc) + timedelta(days=1)).isoformat()
    long_desc = "x" * 501
    result = execute_graphql(f'''
        mutation {{
            addReminder(listId: "{list_id}", text: "Task", description: "{long_desc}", dueAt: "{future_date}") {{
                id
            }}
        }}
    ''')
    
    assert "errors" in result
    assert "500" in result["errors"][0]["message"] or "description" in result["errors"][0]["message"].lower()

def test_add_reminder_past_date():
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    past_date = "2020-01-01T00:00:00Z"
    result = execute_graphql(f'''
        mutation {{
            addReminder(listId: "{list_id}", text: "Task", dueAt: "{past_date}") {{
                id
            }}
        }}
    ''')
    
    assert "errors" in result
    assert "future" in result["errors"][0]["message"].lower()
    
    # Verify no reminder created
    reminders_query = execute_graphql(f'query {{ reminders(listId: "{list_id}") {{ id }} }}')
    assert len(reminders_query["data"]["reminders"]) == 0

def test_reminders_excludes_done_by_default():
    # Setup: list with 2 reminders, one done
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    future1 = (datetime.now(timezone.utc) + timedelta(days=1)).isoformat()
    future2 = (datetime.now(timezone.utc) + timedelta(days=2)).isoformat()
    
    r1 = execute_graphql(f'mutation {{ addReminder(listId: "{list_id}", text: "Task 1", dueAt: "{future1}") {{ id }} }}')
    r2 = execute_graphql(f'mutation {{ addReminder(listId: "{list_id}", text: "Task 2", dueAt: "{future2}") {{ id }} }}')
    
    # Complete first reminder
    execute_graphql(f'mutation {{ completeReminder(id: "{r1["data"]["addReminder"]["id"]}") {{ id }} }}')
    
    # Query without includeDone
    result = execute_graphql(f'query {{ reminders(listId: "{list_id}") {{ id done }} }}')
    reminders = result["data"]["reminders"]
    
    assert len(reminders) == 1
    assert reminders[0]["done"] is False

def test_delete_list_cascades_reminders():
    # Setup: list with reminders
    list_result = execute_graphql('mutation { addList(name: "Home") { id } }')
    list_id = list_result["data"]["addList"]["id"]
    
    future = (datetime.now(timezone.utc) + timedelta(days=1)).isoformat()
    execute_graphql(f'mutation {{ addReminder(listId: "{list_id}", text: "Task", dueAt: "{future}") {{ id }} }}')
    
    # Delete list
    delete_result = execute_graphql(f'mutation {{ deleteList(id: "{list_id}") }}')
    assert delete_result["data"]["deleteList"] is True
    
    # Verify reminders gone
    reminders_result = execute_graphql(f'query {{ reminders(listId: "{list_id}") {{ id }} }}')
    assert len(reminders_result["data"]["reminders"]) == 0
```

---

## Edge Cases

1. **Blank text:** Whitespace-only strings (`"   "`) are treated as empty (rejected per product decision 5).
2. **Blank description:** Empty or whitespace-only description is normalized to None (valid, optional field).
3. **Past dates:** Any `dueAt` ≤ current UTC time is rejected with "dueAt must be in the future" error.
4. **Invalid ISO-8601:** Malformed date strings (e.g. `"not-a-date"`, `"2023-13-01"`) raise parsing error.
5. **Unknown listId:** `addReminder` validates list existence before creating reminder.
6. **Unknown reminder id:** `completeReminder` validates reminder existence before updating.
7. **Cascade delete:** Deleting a list removes all its reminders from `_reminders` and `_list_reminders`; no orphaned reminders.
8. **Sorting edge cases:** Reminders with identical `dueAt` values maintain insertion order (stable sort).
9. **includeDone default:** `reminders` query defaults to `includeDone: false`; done reminders excluded unless explicitly requested.
10. **Description length:** Valid range is 0–500 characters after normalization (blank → None).
11. **Concurrent requests:** In-memory store is not thread-safe; assume single-threaded execution (typical for development GraphQL servers). Production would require locking or atomic operations.

---

## Open Questions

None; all product decisions resolved per requirements. Description field added per review feedback.

---

## Dependencies

- **Python:** 3.10+ (for type hints like `dict | None`)
- **GraphQL library:** Strawberry, Graphene, or Ariadne (same as Story 1)
- **Testing:** pytest + GraphQL test client
- **Story 1 completion:** This story extends `src/schema.py`, `src/resolvers.py`, `src/store.py`, and `src/validation.py` from Story 1

---

## Notes

- **ID generation:** Use `uuid.uuid4()` for unique, non-sequential reminder ids.
- **ISO-8601 parsing:** Use `datetime.fromisoformat()` (Python 3.7+) or `dateutil.parser.isoparse()` for strict parsing.
- **Future validation:** Compare `parse_iso8601(dueAt)` to `datetime.now(timezone.utc)`; reject if ≤.
- **Sorting:** `reminders` query sorts by parsed datetime (oldest first): `sorted(reminders, key=lambda r: parse_iso8601(r["dueAt"]))`.
- **Description handling:** Optional field; blank/empty values normalized to None during validation; stored as None in reminder dict.
- **Cascade implementation:** `delete_list` in `src/store.py` updated to iterate `_list_reminders[list_id]` and remove each reminder before deleting the list.
- **Story 1 compatibility:** All Story 1 operations and tests remain unchanged; `deleteList` behavior transparently extended (no breaking changes).
- **Error messages:** Validation errors include context (e.g. "Reminder text must be between 1 and 200 characters", "Description must not exceed 500 characters", "dueAt must be in the future", "List not found").
