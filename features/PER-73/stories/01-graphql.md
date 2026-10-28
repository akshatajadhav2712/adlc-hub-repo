# LLD — Story 01-graphql: Reminder Lists

**Repo:** `graphql`

**Covers:** AC1, AC2, AC3, AC4, AC9

---

## Acceptance Criteria (from requirements + breakdown)

- **AC1:** addList("Home") returns a list with that name and no reminders; lists includes it.
- **AC2:** addList with an empty, blank, too long or duplicate name is rejected and adds nothing.
- **AC3:** renameList changes the name; an unknown id or a duplicate name is rejected.
- **AC4:** deleteList removes the list and its reminders; an unknown id is rejected.
- **AC9:** health() still returns exactly {"status": "ok"}; existing operations are unchanged.

---

## File Plan

All paths below exist in the repo tree or are new files in established directories.

### New files

- `src/schema.py` — GraphQL schema definitions (types, queries, mutations)
- `src/resolvers.py` — Query and mutation resolver functions
- `src/store.py` — In-memory data structures for lists (and later reminders)
- `src/validation.py` — Validation helpers (name uniqueness, length, blank checks)
- `tests/test_reminder_lists.py` — Tests for this story's operations

### Modified files

- `src/health.py` — No changes; existing health() resolver stays unchanged

### Unchanged files

- `README.md`
- `src/__init__.py`
- `tests/__init__.py`
- `tests/test_health.py`

---

## Data Model

### ReminderList

Represented as a Python dictionary (later formalized as a GraphQL type):

```python
{
    "id": str,          # Unique identifier (UUID v4)
    "name": str,        # 1–50 characters, unique case-insensitive, non-blank
    "reminders": list   # Array of Reminder dicts (empty in this story)
}
```

**Invariants:**
- `id` is unique across all lists
- `name` is unique after normalization (lowercase comparison)
- `name` is non-empty, non-blank (no whitespace-only), ≤50 characters
- `reminders` is an empty list in this story; populated in Story 2

---

## Interfaces

### GraphQL Schema (`src/schema.py`)

```graphql
type ReminderList {
  id: String!
  name: String!
  reminders: [Reminder!]!
}

type Reminder {
  id: String!
  listId: String!
  text: String!
  dueAt: String!
  done: Boolean!
}

type Query {
  lists: [ReminderList!]!
  list(id: String!): ReminderList
  health: HealthStatus!
}

type Mutation {
  addList(name: String!): ReminderList!
  renameList(id: String!, name: String!): ReminderList!
  deleteList(id: String!): Boolean!
}

type HealthStatus {
  status: String!
}
```

**Notes:**
- `Reminder` type is defined now but unused in this story; Story 2 will add reminder operations.
- `health` query preserves existing operation.

---

### Resolver Signatures (`src/resolvers.py`)

```python
# Queries
def resolve_lists(info) -> list[dict]:
    """Returns all reminder lists sorted by name (case-insensitive)."""
    ...

def resolve_list(info, id: str) -> dict | None:
    """Returns one list by id, or None if not found."""
    ...

def resolve_health(info) -> dict:
    """Returns {"status": "ok"}."""
    ...

# Mutations
def resolve_add_list(info, name: str) -> dict:
    """Creates a new list; raises GraphQLError on validation failure."""
    ...

def resolve_rename_list(info, id: str, name: str) -> dict:
    """Renames a list; raises GraphQLError if id unknown or name duplicate."""
    ...

def resolve_delete_list(info, id: str) -> bool:
    """Deletes a list and its reminders; raises GraphQLError if id unknown."""
    ...
```

**Error Handling:**
- Validation failures and unknown ids raise `graphql.GraphQLError` with descriptive messages.
- Errors propagate to the GraphQL response as structured error objects.

---

### In-Memory Store (`src/store.py`)

```python
from typing import Dict, List
from uuid import uuid4

# Module-level storage (cleared on process restart)
_lists: Dict[str, dict] = {}  # keyed by list id
_name_index: Dict[str, str] = {}  # lowercase name -> list id (for uniqueness check)

def get_all_lists() -> List[dict]:
    """Returns all lists sorted by name (case-insensitive)."""
    ...

def get_list_by_id(list_id: str) -> dict | None:
    """Returns a list by id, or None if not found."""
    ...

def add_list(name: str) -> dict:
    """Creates a new list with unique id and empty reminders; raises ValueError on validation failure."""
    ...

def rename_list(list_id: str, new_name: str) -> dict:
    """Updates list name; raises ValueError if id unknown or name duplicate."""
    ...

def delete_list(list_id: str) -> bool:
    """Deletes a list and its reminders; raises ValueError if id unknown. Returns True on success."""
    ...

def _normalize_name(name: str) -> str:
    """Lowercases and strips whitespace for case-insensitive comparison."""
    return name.strip().lower()
```

**Implementation Notes:**
- `_lists` is a dict mapping `id -> list_dict` for O(1) lookup.
- `_name_index` maps `normalized_name -> id` to enforce case-insensitive uniqueness.
- `uuid4()` generates unique list ids.
- Story 2 will add a `_reminders` dict and update `delete_list` to cascade.

---

### Validation Helpers (`src/validation.py`)

```python
def validate_list_name(name: str) -> None:
    """Raises ValueError if name is empty, blank, or exceeds 50 characters."""
    ...

def is_blank(s: str) -> bool:
    """Returns True if string is empty or contains only whitespace."""
    ...
```

**Rules:**
- Empty string (`""`) → ValueError
- Whitespace-only (`"   "`) → ValueError (blank check)
- Length > 50 → ValueError
- Used by `add_list` and `rename_list` before checking uniqueness

---

## Algorithm Notes

### addList(name: String!)

1. Validate name: `validate_list_name(name)` (raises if empty, blank, or >50 chars)
2. Normalize name: `normalized = _normalize_name(name)`
3. Check uniqueness: if `normalized in _name_index`, raise ValueError("List name already exists")
4. Generate id: `list_id = str(uuid4())`
5. Create list dict: `{"id": list_id, "name": name, "reminders": []}`
6. Store: `_lists[list_id] = list_dict`; `_name_index[normalized] = list_id`
7. Return list dict

### renameList(id: String!, name: String!)

1. Validate name: `validate_list_name(name)`
2. Get list: `list_dict = get_list_by_id(id)`; if None, raise ValueError("List not found")
3. Normalize new name: `normalized = _normalize_name(name)`
4. Check if new name conflicts with another list:
   - If `normalized in _name_index` and `_name_index[normalized] != id`, raise ValueError("List name already exists")
5. Remove old name from index: `old_normalized = _normalize_name(list_dict["name"])`; `del _name_index[old_normalized]`
6. Update list: `list_dict["name"] = name`
7. Update index: `_name_index[normalized] = id`
8. Return updated list dict

### deleteList(id: String!)

1. Get list: `list_dict = get_list_by_id(id)`; if None, raise ValueError("List not found")
2. Remove from index: `normalized = _normalize_name(list_dict["name"])`; `del _name_index[normalized]`
3. Delete list: `del _lists[id]`
4. (Story 2 will add: delete all reminders where `listId == id`)
5. Return True

### lists

1. Fetch all: `all_lists = list(_lists.values())`
2. Sort by name (case-insensitive): `sorted(all_lists, key=lambda l: l["name"].lower())`
3. Return sorted list

### list(id: String!)

1. Return `_lists.get(id, None)` (None if not found)

### health()

1. Return `{"status": "ok"}` (unchanged from existing implementation in `src/health.py`)

---

## Tests

**File:** `tests/test_reminder_lists.py`

**Test count: 13** (matches breakdown Verification Approach count)

### Test cases (mapped to requirements)

1. **REQ-1 / AC1 (partial):** `test_lists_empty_initially` — Query `lists` returns empty array on startup
2. **REQ-1 / AC1 (partial):** `test_lists_includes_added_list` — After `addList(name: "Home")`, `lists` includes it sorted by name
3. **REQ-2 / AC1 (partial):** `test_list_by_id_valid` — `list(id: "<valid>")` returns the list with that id
4. **REQ-2 / AC1 (partial):** `test_list_by_id_unknown` — `list(id: "unknown")` returns null
5. **REQ-3 / AC1:** `test_add_list_success` — `addList(name: "Home")` creates list with unique id, name "Home", empty reminders array
6. **REQ-4 / AC2 (empty):** `test_add_list_empty_name` — `addList(name: "")` raises GraphQLError, adds nothing
7. **REQ-4 / AC2 (blank):** `test_add_list_blank_name` — `addList(name: "   ")` raises GraphQLError, adds nothing
8. **REQ-4 / AC2 (over limit):** `test_add_list_name_too_long` — `addList(name: "<51-char string>")` raises GraphQLError, adds nothing
9. **REQ-4 / AC2 (duplicate exact):** `test_add_list_duplicate_name_exact` — `addList(name: "Home")` when "Home" exists raises GraphQLError, adds nothing
10. **REQ-4 / AC2 (duplicate case-insensitive):** `test_add_list_duplicate_name_case_insensitive` — `addList(name: "home")` when "Home" exists raises GraphQLError, adds nothing
11. **REQ-5 / AC3 (success):** `test_rename_list_success` — `renameList(id: "<valid>", name: "Work")` updates name and returns updated list
12. **REQ-6 / AC3 (unknown id):** `test_rename_list_unknown_id` — `renameList(id: "unknown", name: "Work")` raises GraphQLError, modifies nothing
13. **REQ-6 / AC3 (duplicate name):** `test_rename_list_duplicate_name` — `renameList(id: "<valid>", name: "Home")` when another list named "Home" exists raises GraphQLError, modifies nothing

**Additional tests covering REQ-7, REQ-8, REQ-14 (AC4, AC9):**

14. **REQ-7, REQ-8 / AC4 (success):** `test_delete_list_success` — `deleteList(id: "<valid>")` removes list and returns True
15. **REQ-7, REQ-8 / AC4 (unknown id):** `test_delete_list_unknown_id` — `deleteList(id: "unknown")` raises GraphQLError, modifies nothing
16. **REQ-14 / AC9:** `test_health_unchanged` — `health()` returns exactly `{"status": "ok"}`

**Corrected test count: 16 tests total** (breakdown listed 13 as target; this LLD provides complete 1:1 AC coverage including all edge cases from requirements).

### Test structure

Each test:
1. Sets up initial state (add lists if needed)
2. Executes GraphQL query or mutation
3. Asserts expected result or error
4. Verifies no side effects (e.g. lists count unchanged on rejection)

**Example (pseudo-code):**

```python
def test_add_list_success():
    result = execute_graphql('mutation { addList(name: "Home") { id name reminders } }')
    assert result["data"]["addList"]["name"] == "Home"
    assert result["data"]["addList"]["reminders"] == []
    assert len(execute_graphql('query { lists { id } }')["data"]["lists"]) == 1

def test_add_list_duplicate_name_case_insensitive():
    execute_graphql('mutation { addList(name: "Home") { id } }')
    result = execute_graphql('mutation { addList(name: "home") { id } }')
    assert "errors" in result
    assert "already exists" in result["errors"][0]["message"].lower()
    assert len(execute_graphql('query { lists { id } }')["data"]["lists"]) == 1  # no second list
```

---

## Edge Cases

1. **Blank names:** Whitespace-only strings (`"   "`) are treated as empty (rejected per product decision 5).
2. **Case-insensitive uniqueness:** "Home" and "home" are duplicates; normalized comparison prevents both from existing.
3. **Unknown id handling:** All mutations that take an `id` (renameList, deleteList) validate existence before operating; return meaningful errors.
4. **Cascade delete (Story 2):** This story's `deleteList` has no reminders to delete; Story 2 will extend the function.
5. **Concurrent requests:** In-memory store is not thread-safe; assume single-threaded execution (typical for development GraphQL servers). Production would require locking or atomic operations.

---

## Open Questions

None; all product decisions resolved per requirements.

---

## Dependencies

- **Python:** 3.10+ (for type hints like `dict | None`)
- **GraphQL library:** Strawberry, Graphene, or Ariadne (choose based on existing `src/health.py` implementation; likely Strawberry or Graphene given Python ecosystem)
- **Testing:** pytest + GraphQL test client (e.g. `strawberry.test.client` or `graphene.test.Client`)

---

## Notes

- **ID generation:** Use `uuid.uuid4()` for unique, non-sequential ids.
- **Sorting:** `lists` query sorts by `name.lower()` for case-insensitive alphabetical order.
- **Error messages:** Validation errors include context (e.g. "List name must be between 1 and 50 characters", "List name already exists").
- **Story 2 impact:** The `deleteList` resolver will be extended to cascade-delete reminders; `_reminders` dict will be added to `src/store.py`.
