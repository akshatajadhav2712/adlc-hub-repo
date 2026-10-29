# High-Level Design — Add an in-memory GraphQL todo list

**Story ID:** PER-80  
**Epic:** N/A (standalone story)  
**Repository:** graphql  
**Date:** 2025-01-19

---

## Overview

This design adds an in-memory todo list feature to the existing `graphql` service without modifying the existing `health()` query. The todo list lives entirely in memory and resets on process restart. No database, authentication, or UI is required.

---

## Architecture Decisions

### AD-1: Enum-based done field

**Binds:** The `done` field is defined as a GraphQL enum type `TodoStatus` with values `DONE` and `NOT_DONE`.

**Prevents:** Boolean ambiguity in GraphQL responses and ensures explicit status representation.

**Rule:** All todo mutations and queries return `done` as a `TodoStatus` enum value, not a boolean. The in-memory storage uses the enum directly.

**Status:** `[ADOPTED]` — per review feedback (architect_review)

---

### AD-2: In-memory module-level storage

**Binds:** Todo list storage is a module-level list in `src/todo.py`, no external state management.

**Prevents:** Introducing database, file system, or persistence dependencies outside the ticket scope.

**Rule:** The list `_todos: List[Todo] = []` is initialized once at module load and mutated by business logic functions. No thread safety is implemented.

**Status:** `[ADOPTED]` — ticket constraint

---

### AD-3: Health endpoint isolation

**Binds:** The existing `health()` query and `src/health.py` module remain completely untouched.

**Prevents:** Accidental regression of the health endpoint behavior.

**Rule:** No changes to `src/health.py`, `tests/test_health.py`, or the health resolver. The GraphQL schema extends existing types without modifying them.

**Status:** `[ADOPTED]` — ticket constraint (AC6)

---

### AD-4: UUID-based ID generation

**Binds:** Todo IDs are generated using Python's `uuid.uuid4()` and stored as strings.

**Prevents:** ID collision and reliance on incremental counters that complicate testing.

**Rule:** Each `add_todo()` call generates a new UUID string for the `id` field.

**Status:** `[ADOPTED]` — simple, deterministic, no external dependency

---

## Components

### 1. Business Logic (`src/todo.py`)

**Purpose:** Manages the in-memory todo list and enforces business rules.

**Exports:**
- `TodoStatus` enum: `DONE`, `NOT_DONE`
- `Todo` dataclass: `id: str`, `title: str`, `done: TodoStatus`
- `add_todo(title: str) -> Todo`
- `get_all_todos() -> List[Todo]`
- `complete_todo(id: str) -> Todo`

**Behaviors:**
- `add_todo(title)` validates `title` is non-empty (raises `ValueError` if empty), generates UUID, appends to `_todos`, returns the new todo with `done=NOT_DONE`.
- `get_all_todos()` returns a copy of the list.
- `complete_todo(id)` finds the todo by `id`, sets `done=DONE`, returns the updated todo (raises `ValueError` if not found).

**Dependencies:** Python standard library (`uuid`, `dataclasses`).

---

### 2. GraphQL Schema (`src/schema.py`)

**Purpose:** Defines the GraphQL type system combining health and todo operations.

**Schema Extensions:**

```graphql
enum TodoStatus {
  DONE
  NOT_DONE
}

type Todo {
  id: String!
  title: String!
  done: TodoStatus!
}

type Query {
  health: HealthStatus!
  todos: [Todo!]!
}

type Mutation {
  addTodo(title: String!): Todo!
  completeTodo(id: String!): Todo!
}
```

**Behaviors:**
- Existing `health` query preserved unchanged.
- `todos` query returns all in-memory todos.
- `addTodo` mutation creates and returns a todo.
- `completeTodo` mutation updates and returns a todo.

**Dependencies:** GraphQL library already in the project (inferred from existing `health()` implementation).

---

### 3. Resolvers (`src/resolvers.py`)

**Purpose:** Connects GraphQL operations to business logic.

**Resolver Functions:**
- `resolve_todos() -> List[Todo]` — calls `get_all_todos()`
- `resolve_add_todo(title: str) -> Todo` — calls `add_todo(title)`, catches `ValueError` and translates to GraphQL error
- `resolve_complete_todo(id: str) -> Todo` — calls `complete_todo(id)`, catches `ValueError` and translates to GraphQL error

**Behaviors:**
- Empty title input to `addTodo` → GraphQL error response, no todo added (AC4)
- Unknown ID input to `completeTodo` → GraphQL error response, no todos modified (AC5)
- All errors surfaced as user-facing GraphQL errors, not server crashes.

**Dependencies:** `src/todo.py`, GraphQL library.

---

### 4. Existing Health Module (`src/health.py`)

**Purpose:** Provides health check endpoint.

**Status:** **Untouched** — no changes to this file or its tests.

---

## Data Flow

### Add Todo Flow (AC1, AC4)

```
Client GraphQL request: addTodo(title: "Buy milk")
  ↓
resolve_add_todo(title="Buy milk")
  ↓
add_todo(title="Buy milk")
  → validate title non-empty
  → generate UUID
  → create Todo(id=uuid, title="Buy milk", done=NOT_DONE)
  → append to _todos
  ↓
return Todo to resolver
  ↓
GraphQL response: { id, title, done: NOT_DONE }
```

**Error case:** `addTodo(title: "")` → `add_todo("")` raises `ValueError` → resolver catches and returns GraphQL error.

---

### Query Todos Flow (AC2)

```
Client GraphQL request: todos
  ↓
resolve_todos()
  ↓
get_all_todos()
  → return copy of _todos
  ↓
GraphQL response: [{ id, title, done }, ...]
```

---

### Complete Todo Flow (AC3, AC5)

```
Client GraphQL request: completeTodo(id: "abc-123")
  ↓
resolve_complete_todo(id="abc-123")
  ↓
complete_todo(id="abc-123")
  → find todo in _todos by id
  → if found: set done=DONE, return todo
  → if not found: raise ValueError
  ↓
return Todo to resolver (or catch error)
  ↓
GraphQL response: { id, title, done: DONE } (or error)
```

---

## File Structure

### New Files

- **`src/todo.py`** — Business logic: `Todo` dataclass, `TodoStatus` enum, in-memory list, `add_todo()`, `get_all_todos()`, `complete_todo()`
- **`src/schema.py`** — GraphQL schema definition extending existing health schema with todo types, queries, and mutations
- **`src/resolvers.py`** — Resolver functions mapping GraphQL operations to `src/todo.py` functions
- **`tests/test_todo.py`** — Unit tests for business logic (5 test cases covering add, get, complete, validation errors)
- **`tests/test_resolvers.py`** — Integration tests for GraphQL resolvers (6 test cases mapping to ACs 1–6)

### Unchanged Files

- **`src/health.py`** — Health endpoint logic (no changes)
- **`tests/test_health.py`** — Health endpoint tests (no changes)
- **`src/__init__.py`** — Module initialization (may need to import new schema/resolvers if not auto-discovered)
- **`tests/__init__.py`** — Test module initialization (no changes needed)

---

## Risks and Mitigations

### Risk 1: GraphQL library schema registration
**Description:** The existing `health()` query may be registered in a way that doesn't support extending the schema with new types and resolvers.

**Mitigation:** Investigate the current GraphQL setup in `src/health.py` early. If the library supports schema merging (e.g., Strawberry, Ariadne, Graphene), extend the schema in `src/schema.py`. If not, refactor the schema registration to a shared entry point before adding todo operations.

**Impact if unmitigated:** Schema extension blocked, requires rework.

---

### Risk 2: Enum representation in responses
**Description:** GraphQL enum values may serialize as strings (`"DONE"`) or other representations depending on the library.

**Mitigation:** Verify the enum output format in integration tests (`tests/test_resolvers.py`) and document the expected response format. Ensure the Python `TodoStatus` enum values map correctly to GraphQL enum values.

**Impact if unmitigated:** Client confusion or test failures if responses don't match expectations.

---

### Risk 3: Thread safety
**Description:** The in-memory `_todos` list is not thread-safe. Concurrent requests could cause race conditions.

**Mitigation:** Accept this limitation per the ticket's demo scope. Document in code comments that the implementation is not production-ready. If the service uses async (e.g., async resolvers), ensure the list mutations complete atomically within a single request.

**Impact if unmitigated:** Rare race conditions in concurrent testing; acceptable for demo scope.

---

## Verification Strategy

### Unit Tests (`tests/test_todo.py`)
- Test `add_todo("Buy milk")` returns `Todo` with correct `id`, `title="Buy milk"`, `done=NOT_DONE`
- Test `add_todo("")` raises `ValueError`
- Test `get_all_todos()` after adding one todo returns list with that todo
- Test `complete_todo(valid_id)` sets `done=DONE` and returns updated todo
- Test `complete_todo("invalid-id")` raises `ValueError`

### Integration Tests (`tests/test_resolvers.py`)
Map directly to ACs:
- **AC1:** GraphQL `addTodo(title: "Buy milk")` returns `{ id, title: "Buy milk", done: NOT_DONE }`
- **AC2:** GraphQL `todos` query after `addTodo` includes the added todo
- **AC3:** GraphQL `completeTodo(id: "<valid-id>")` returns `{ id, title, done: DONE }`
- **AC4:** GraphQL `addTodo(title: "")` returns error, `todos` query shows no new todo
- **AC5:** GraphQL `completeTodo(id: "nonexistent")` returns error, `todos` query unchanged
- **AC6:** GraphQL `health` query returns exactly `{"status": "ok"}`

---

## Open Questions

None — design is fully specified per the ticket and review feedback.

---

## Deferred Decisions

- **Thread safety implementation:** Not required for demo scope; in-memory list is not thread-safe.
- **Todo deletion or editing:** Not in scope for this story.
- **Persistent storage:** Explicitly out of scope; list resets on process restart.
- **Authentication/authorization:** Not required.
- **Advanced todo features:** Due dates, priorities, categories, etc., are out of scope.

---

## Success Criteria

The design is successfully implemented when:
1. All six acceptance criteria (AC1–AC6) pass in integration tests.
2. The `health()` query continues to return exactly `{"status": "ok"}`.
3. The `done` field in all responses is a `TodoStatus` enum (`DONE` or `NOT_DONE`), not a boolean.
4. All code is contained in the `graphql` repository.
5. No changes to `src/health.py` or `tests/test_health.py`.
