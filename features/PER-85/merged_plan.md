# Merged plan — PER-85

Execution order (1 ticket(s)):

1. `01`

---

## Ticket `01`

# Technical Specification — Story 01: Add an in-memory GraphQL todo list

## Metadata

```yaml
story_id: 01
ticket: PER-85
repository: graphql
stack: Python (existing codebase)
complexity: simple
status: draft
```

## Stack

**Language:** Python  
**Framework:** None (standard library only)  
**Repository:** `graphql` at `/home/aksj/Documents/adlc-graphql`  
**Existing structure:**
- `src/__init__.py`, `src/health.py`
- `tests/__init__.py`, `tests/test_health.py`

**Rationale:** The repo is already Python; no additional dependencies are required for in-memory storage and basic operations.

## Data Model

### Todo

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | string | Yes (generated) | — | Unique identifier |
| `title` | string | Yes | — | Todo description |
| `done` | boolean | No | `false` | Completion status |
| `description` | string | No | `None` | Optional additional detail |

**ID generation:** UUID4 string (via `uuid.uuid4()`)  
**Representation:** Python `dict` with keys `id`, `title`, `done`, `description`

**Example:**
```python
{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "title": "Buy milk",
    "done": False,
    "description": None
}
```

## API Contracts

### Query: `todos() -> list[dict]`

**Returns:** List of all todos in memory

**Output format:**
```python
[
    {"id": "...", "title": "Buy milk", "done": False, "description": None},
    {"id": "...", "title": "Walk dog", "done": True, "description": "In the park"}
]
```

**Behavior:** Returns every todo currently in the in-memory list. Empty list `[]` if no todos exist.

---

### Mutation: `add_todo(title: str) -> dict`

**Input:**
- `title` (required, non-empty string)

**Returns:** The newly created todo (dict)

**Output format:**
```python
{"id": "abc-123", "title": "Buy milk", "done": False, "description": None}
```

**Behavior:**
1. Validate `title` is non-empty (not `""`)
2. Generate unique `id` via `uuid.uuid4()`
3. Create todo dict with `done=False` and `description=None`
4. Append to in-memory list
5. Return the created todo

**Error cases:**
- `title == ""` → Raise `ValueError("Title cannot be empty")`, add nothing

---

### Mutation: `complete_todo(id: str) -> dict`

**Input:**
- `id` (required, string)

**Returns:** The updated todo (dict)

**Output format:**
```python
{"id": "abc-123", "title": "Buy milk", "done": True, "description": None}
```

**Behavior:**
1. Find todo by `id` in the in-memory list
2. Set `done=True` (mutate in place)
3. Return the updated todo

**Error cases:**
- `id` not found → Raise `ValueError(f"Todo with id {id} not found")`, modify nothing

---

### Query: `health() -> dict`

**Returns:** `{"status": "ok"}` (unchanged from existing implementation)

**File:** `src/health.py` (untouched)

## File Touch List

### Files to create

1. **`src/todo.py`**
   - **Purpose:** In-memory todo list storage and operations
   - **Exports:**
     - `TODO_LIST: list[dict]` (module-level, initially `[]`)
     - `add_todo(title: str) -> dict`
     - `complete_todo(id: str) -> dict`
     - `get_todos() -> list[dict]`
   - **Lines of code (est.):** ~40

2. **`tests/test_todo.py`**
   - **Purpose:** Test all six acceptance criteria
   - **Test functions:**
     - `test_add_todo_with_valid_title()` (AC1)
     - `test_query_returns_added_todos()` (AC2)
     - `test_complete_todo_by_id()` (AC3)
     - `test_reject_empty_title()` (AC4)
     - `test_reject_unknown_id_for_completion()` (AC5)
     - `test_health_query_unchanged()` (AC6)
   - **Lines of code (est.):** ~60

### Files unchanged

- `src/__init__.py` (no changes needed)
- `src/health.py` (existing health check, untouched)
- `tests/__init__.py` (no changes needed)
- `tests/test_health.py` (existing health tests, untouched)

## Implementation Details

### `src/todo.py`

```python
import uuid

TODO_LIST = []

def get_todos():
    """Return all todos."""
    return TODO_LIST

def add_todo(title):
    """Create and store a new todo. Raise ValueError if title is empty."""
    if not title or title == "":
        raise ValueError("Title cannot be empty")
    
    todo = {
        "id": str(uuid.uuid4()),
        "title": title,
        "done": False,
        "description": None
    }
    TODO_LIST.append(todo)
    return todo

def complete_todo(id):
    """Mark a todo as done. Raise ValueError if id not found."""
    for todo in TODO_LIST:
        if todo["id"] == id:
            todo["done"] = True
            return todo
    raise ValueError(f"Todo with id {id} not found")
```

### `tests/test_todo.py`

```python
import pytest
from src.todo import TODO_LIST, add_todo, complete_todo, get_todos
from src.health import health

def setup_function():
    """Clear todo list before each test."""
    TODO_LIST.clear()

def test_add_todo_with_valid_title():
    """AC1: addTodo("Buy milk") returns todo with that title and done=False."""
    todo = add_todo("Buy milk")
    assert todo["title"] == "Buy milk"
    assert todo["done"] is False

def test_query_returns_added_todos():
    """AC2: todos includes the added todo."""
    todo = add_todo("Buy milk")
    todos = get_todos()
    assert todo in todos

def test_complete_todo_by_id():
    """AC3: completeTodo returns the same todo with done=True."""
    todo = add_todo("Buy milk")
    updated = complete_todo(todo["id"])
    assert updated["id"] == todo["id"]
    assert updated["done"] is True

def test_reject_empty_title():
    """AC4: addTodo("") is rejected and adds nothing."""
    initial_count = len(TODO_LIST)
    with pytest.raises(ValueError, match="Title cannot be empty"):
        add_todo("")
    assert len(TODO_LIST) == initial_count

def test_reject_unknown_id_for_completion():
    """AC5: completeTodo with unknown id is rejected."""
    with pytest.raises(ValueError, match="not found"):
        complete_todo("nonexistent-id")

def test_health_query_unchanged():
    """AC6: health() still returns exactly {"status": "ok"}."""
    result = health()
    assert result == {"status": "ok"}
```

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| In-memory list grows unbounded | Low | Low | Acceptable for demo scope; no production requirement |
| Concurrent access race conditions | Low | Low | Single-process assumption; no concurrency in ticket |
| ID collision (UUID4) | Very Low | Medium | UUID4 collision probability is negligible for demo scale |
| Breaking `health()` accidentally | Low | High | Existing `test_health.py` will catch regressions; run before commit |

## Verification

Each AC maps to one test function in `tests/test_todo.py`:

1. **AC1** → `test_add_todo_with_valid_title`: Call `add_todo("Buy milk")`, assert returned todo has `title=="Buy milk"` and `done==False`
2. **AC2** → `test_query_returns_added_todos`: Add todo, call `get_todos()`, assert list includes the added todo
3. **AC3** → `test_complete_todo_by_id`: Add todo, capture `id`, call `complete_todo(id)`, assert returned todo has `done==True`
4. **AC4** → `test_reject_empty_title`: Call `add_todo("")`, assert `ValueError` raised, verify list count unchanged
5. **AC5** → `test_reject_unknown_id_for_completion`: Call `complete_todo("nonexistent-id")`, assert `ValueError` raised
6. **AC6** → `test_health_query_unchanged`: Call `health()`, assert response is exactly `{"status": "ok"}`, run existing `tests/test_health.py`

**Test execution:**
```bash
pytest tests/test_todo.py -v
pytest tests/test_health.py -v  # verify no regressions
```

## Notes

- The in-memory list (`TODO_LIST`) resets when the process restarts (module-level variable)
- No external dependencies required (uses only `uuid` from standard library)
- `src/health.py` and `tests/test_health.py` remain byte-for-byte identical
- Optional `description` field added per review feedback; defaults to `None`, no validation

## References

- **Ticket:** PER-85
- **Requirements:** `requirements.md`
- **HLD:** `hld.md`
- **Story:** `stories/01-graphql.md`
- **Review feedback:** `review_feedback.md` (architect_review, plan v2)

---

# Story summaries

- `01-graphql`: Story — PER-85: Add an in-memory GraphQL todo list

---

_Single-ticket pack: order is trivial (no multi-split)._
