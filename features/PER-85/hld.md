# High-Level Design — PER-85: Add an in-memory GraphQL todo list

## Overview

This feature extends the existing `graphql` service with an in-memory todo list. The service already exposes a `health()` query that returns `{"status": "ok"}`; that query remains unchanged. The new todo list lives in memory only and is reset when the process restarts.

**Repository:** `graphql` at `/home/aksj/Documents/adlc-graphql`  
**Language/Stack:** Python (matches existing `src/health.py`)  
**Scope:** Single-repo feature with no external dependencies, databases, or UI

## Architecture

### Component Structure

```
graphql/
├── src/
│   ├── __init__.py          [existing]
│   ├── health.py            [existing, unchanged]
│   └── todo.py              [new]
└── tests/
    ├── __init__.py          [existing]
    ├── test_health.py       [existing, unchanged]
    └── test_todo.py         [new]
```

### Components

#### `src/todo.py` (new)

**Responsibility:** In-memory todo list storage and operations

**Exports:**
- `Todo` — Data class or dictionary structure representing a todo item
- `add_todo(title: str) -> Todo` — Creates and stores a new todo
- `complete_todo(id: str) -> Todo` — Marks a todo as done
- `get_todos() -> list[Todo]` — Returns all todos

**Internal state:**
- In-memory collection (list or dict) holding all todos
- Resets to empty when process restarts

**Validation logic:**
- `add_todo`: rejects empty string (`""`) for title
- `complete_todo`: rejects unknown `id`

#### `tests/test_todo.py` (new)

**Responsibility:** Test coverage for all six acceptance criteria

**Test cases:**
1. `test_add_todo_with_valid_title` (AC1)
2. `test_query_returns_added_todos` (AC2)
3. `test_complete_todo_by_id` (AC3)
4. `test_reject_empty_title` (AC4)
5. `test_reject_unknown_id_for_completion` (AC5)
6. `test_health_query_unchanged` (AC6)

#### `src/health.py` (existing, unchanged)

**Responsibility:** Existing health check

**Behavior:** Returns `{"status": "ok"}` exactly as before

## Data Model

### Todo

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | string | Yes (generated) | — | Unique identifier |
| `title` | string | Yes | — | Todo description |
| `done` | boolean | No | `false` | Completion status |
| `description` | string | No | `None` or `""` | Optional additional detail |

**ID generation:** Implementation may use UUID, incremental counter, or timestamp-based string. Choice is deferred to implementation.

**Representation:** Python dict or dataclass. Choice is deferred to implementation.

## API / Operations

### Query: `todos`

**Returns:** List of all todos

**Output format:**
```python
[
    {"id": "...", "title": "Buy milk", "done": False, "description": "..."},
    {"id": "...", "title": "Walk dog", "done": True, "description": "..."},
    ...
]
```

**Behavior:** Returns every todo currently in the in-memory list.

### Mutation: `addTodo(title: String!)`

**Input:**
- `title` (required, non-empty string)

**Returns:** The newly created todo

**Output format:**
```python
{"id": "abc123", "title": "Buy milk", "done": False, "description": None}
```

**Behavior:**
1. Validate `title` is non-empty
2. Generate unique `id`
3. Create todo with `done=False` and optional `description=None`
4. Add to in-memory list
5. Return the created todo

**Error cases:**
- Empty string (`""`) → Reject with error, add nothing

### Mutation: `completeTodo(id: String!)`

**Input:**
- `id` (required, string)

**Returns:** The updated todo

**Output format:**
```python
{"id": "abc123", "title": "Buy milk", "done": True, "description": None}
```

**Behavior:**
1. Find todo by `id`
2. Set `done=True`
3. Return the updated todo

**Error cases:**
- Unknown `id` → Reject with error, modify nothing

## State Management

**Storage:** In-memory Python data structure (list or dict)

**Lifecycle:** Resets to empty when process restarts

**Concurrency:** Not addressed (single-process, in-memory, no concurrent access requirements in ticket)

## Error Handling

| Scenario | Response | Invariant |
|----------|----------|-----------|
| `addTodo("")` | Error (validation failure) | No todo added to list |
| `completeTodo("unknown-id")` | Error (not found) | No todos modified |
| `health()` | `{"status": "ok"}` | Unchanged from existing behavior |

**Error format:** Implementation-specific (exception, error dict, or None return). Choice is deferred to implementation.

## Verification Strategy

Each acceptance criterion maps to one test case:

1. **AC1** → `test_add_todo_with_valid_title`: Call `add_todo("Buy milk")`, assert returned todo has `title=="Buy milk"` and `done==False`
2. **AC2** → `test_query_returns_added_todos`: Add todo, call `get_todos()`, assert list includes the added todo
3. **AC3** → `test_complete_todo_by_id`: Add todo, capture `id`, call `complete_todo(id)`, assert returned todo has `done==True`
4. **AC4** → `test_reject_empty_title`: Call `add_todo("")`, assert error raised/returned, verify list count unchanged
5. **AC5** → `test_reject_unknown_id_for_completion`: Call `complete_todo("nonexistent-id")`, assert error raised/returned, verify no todos modified
6. **AC6** → `test_health_query_unchanged`: Call existing `health()`, assert response is exactly `{"status": "ok"}`, run existing `tests/test_health.py` and verify all pass

## Architecture Decisions

### AD-1: In-memory storage only

**Binds:** Todo list stored in module-level Python variable (list or dict)

**Prevents:** Feature diverging into database, file system, or external cache requirements

**Rule:** No persistent storage; list resets on process restart

**Status:** `[ADOPTED]` (explicit ticket requirement)

### AD-2: No changes to health endpoint

**Binds:** `src/health.py` and `tests/test_health.py` remain byte-for-byte identical

**Prevents:** Accidental regression of existing working functionality

**Rule:** Health check returns exactly `{"status": "ok"}` forever

**Status:** `[ADOPTED]` (explicit ticket requirement)

### AD-3: Validation before mutation

**Binds:** Input validation happens before any state change

**Prevents:** Invalid data entering the todo list

**Rule:** 
- `add_todo`: Reject empty title before creating todo
- `complete_todo`: Verify `id` exists before modifying

**Status:** `[ADOPTED]` (explicit AC4 and AC5 requirements)

### AD-4: Optional description field

**Binds:** Todo data model includes `description` field (string, optional)

**Prevents:** Future requirement conflicts if description is needed later

**Rule:** Field may be `None`, empty string, or omitted; no validation on description content

**Status:** `[ADOPTED]` (review feedback: "Add Description field with data type as string and that should be optional")

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| In-memory list grows unbounded | Low | Low | Acceptable for demo; no production scale requirement |
| Concurrent access race conditions | Low | Low | Single-process assumption from ticket; no concurrency requirement |
| ID collision | Low | Medium | Use UUID or similar collision-resistant generator |

## Deferred Decisions

- **ID generation strategy** (UUID vs counter vs timestamp) — deferred to implementation
- **Data structure choice** (list vs dict for storage) — deferred to implementation
- **Error format** (exception vs error dict vs None) — deferred to implementation
- **Description field default value** (`None` vs `""`) — deferred to implementation

## Implementation Notes

- Python project at `/home/aksj/Documents/adlc-graphql`
- Existing structure: `src/__init__.py`, `src/health.py`, `tests/test_health.py`
- No external dependencies required beyond Python standard library
- No GraphQL schema file mentioned in repo tree; if schema exists elsewhere, it will need updating (out of scope for this HLD)

## References

- **Ticket:** PER-85
- **Requirements:** `requirements.md` (this feature)
- **Story:** `stories/01-graphql.md`
- **Review feedback:** `review_feedback.md` (architect_review, plan v2)
