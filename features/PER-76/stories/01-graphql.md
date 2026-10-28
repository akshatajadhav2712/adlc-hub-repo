# Story — Add an in-memory GraphQL todo list

**ID:** PER-76  
**Type:** Story  
**Epic:** (none)  
**Status:** Draft  
**Assignee:** (unassigned)

---

## Summary

Extend the existing `graphql` service with a minimal in-memory todo list. The service already exposes `health()`; this story adds three GraphQL operations (`todos`, `addTodo`, `completeTodo`) and an in-memory store. No database, no UI, no authentication. The list resets when the process restarts.

---

## Acceptance Criteria

### AC1: Add a todo with valid title

**Given** the todo list is empty  
**When** I call `addTodo(title: "Buy milk")`  
**Then** it returns a todo with `title = "Buy milk"`, `done = false`, and a unique `id`

### AC2: Query all todos

**Given** I have added a todo with title "Buy milk"  
**When** I call `todos`  
**Then** the result includes that todo

### AC3: Complete an existing todo

**Given** a todo with id `<known-id>` exists and `done = false`  
**When** I call `completeTodo(id: "<known-id>")`  
**Then** it returns the same todo with `done = true`

### AC4: Reject empty title

**Given** the todo list is in any state  
**When** I call `addTodo(title: "")`  
**Then** the operation is rejected and no todo is added

### AC5: Reject unknown id on complete

**Given** a todo with id `"xyz"` does not exist  
**When** I call `completeTodo(id: "xyz")`  
**Then** the operation is rejected

### AC6: Health endpoint unchanged

**Given** the service is running  
**When** I call `health()`  
**Then** it returns exactly `{"status": "ok"}` with no side effects

---

## Technical Specification

### Overview

Add in-memory todo list functionality to the `graphql` service. The existing `src/health.py` module remains untouched. All new code lives in new modules under `src/`.

### GraphQL Schema

Define the following in a new file `src/schema.py`:

```python
type Todo {
  id: String!
  title: String!
  done: Boolean!
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

### In-Memory Storage

Create `src/store.py`:

- Data structure: `dict[str, dict]` keyed by todo `id`, where each value is `{"id": str, "title": str, "done": bool}`.
- Module-level variable: `_todos: dict[str, dict] = {}` (resets when the process restarts).
- Helper functions:
  - `get_all_todos() -> list[dict]`: return `list(_todos.values())`.
  - `add_todo(title: str) -> dict`: generate unique `id` (use `uuid.uuid4().hex`), validate `title` is non-empty, store, return the todo dict.
  - `complete_todo(id: str) -> dict`: validate `id` exists in `_todos`, set `done = True`, return the updated todo dict.
- Validation errors: raise `ValueError` with a descriptive message (resolver layer will convert to GraphQL error).

### Resolvers

Create `src/resolvers.py`:

- **Query.todos**: call `store.get_all_todos()`, return result.
- **Mutation.addTodo**: call `store.add_todo(title)`, catch `ValueError` and re-raise as GraphQL user error, return result on success.
- **Mutation.completeTodo**: call `store.complete_todo(id)`, catch `ValueError` and re-raise as GraphQL user error, return result on success.

### Integration

- `src/schema.py` imports and wires the resolvers to the schema.
- `src/health.py` is unchanged; the existing `health()` query remains as-is.

### File Changes

| File | Action | Notes |
|------|--------|-------|
| `src/schema.py` | create | GraphQL type defs and resolver bindings |
| `src/store.py` | create | In-memory storage and business logic |
| `src/resolvers.py` | create | GraphQL resolver implementations |
| `src/health.py` | unchanged | Existing health check stays intact |
| `tests/test_todos.py` | create | Unit tests covering all six ACs |
| `tests/test_health.py` | unchanged | Existing health tests remain |

---

## Tests

All six acceptance criteria map directly to test cases in `tests/test_todos.py`:

1. **test_add_todo_with_valid_title** — AC1: call `addTodo("Buy milk")`, assert returned todo has correct `title`, `done=False`, and non-empty `id`.
2. **test_query_todos_includes_added** — AC2: add a todo, query `todos`, assert the list includes it.
3. **test_complete_existing_todo** — AC3: add a todo, call `completeTodo(id)`, assert returned todo has `done=True`.
4. **test_reject_empty_title** — AC4: call `addTodo("")`, assert error raised, query `todos` and confirm list is unchanged.
5. **test_reject_unknown_id** — AC5: call `completeTodo("xyz")`, assert error raised.
6. **test_health_unchanged** — AC6: call `health()`, assert response is `{"status": "ok"}`.

Additional setup:
- Each test clears `store._todos` before running (use a fixture or `setUp` method).
- Tests use the GraphQL executor to invoke operations, not direct function calls.

---

## Dependencies

None.

---

## Out of Scope

- Persistent storage (database, file)
- Authentication or authorization
- Frontend UI
- Editing or deleting todos
- Changes to the `agent` or `mcp` repos

---

## Notes

- **ID generation:** `uuid.uuid4().hex` produces unique string IDs; no collision risk in memory-only scope.
- **Validation approach:** Business logic raises `ValueError`; resolvers catch and convert to GraphQL errors.
- **Process restart:** The in-memory store is deliberately ephemeral; no migration or persistence logic is needed.

---

## Open Questions

None.
