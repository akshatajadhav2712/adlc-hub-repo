# Story PER-78 — Add an in-memory GraphQL todo list

**Type:** Story  
**Status:** Draft  
**Repository:** `graphql`  
**Complexity:** Simple  
**Track:** Quick

---

## Overview

Add a small in-memory todo list to the existing GraphQL service. The service already exposes `health()`, which remains unchanged. The todo list state lives in memory only and resets when the process restarts.

---

## Scope

**In scope:**
- In-memory todo storage (no persistence)
- GraphQL schema for todo type and operations
- Three GraphQL operations: `todos`, `addTodo`, `completeTodo`
- Validation for empty title and unknown ID

**Out of scope:**
- Database or file persistence
- Authentication or authorization
- User interface
- Changes to `agent` or `mcp` repos
- Changes to existing `health()` operation

---

## Technical Specification

### Data Model

A todo has:
- `id`: string (unique identifier, generated on creation)
- `title`: string (non-empty)
- `done`: boolean (default `false`)

### Storage

Use an in-memory data structure (e.g., Python `list` or `dict`) to hold todos. The list is empty when the process starts and resets on restart.

### GraphQL Schema

Define a `Todo` type and three operations in the existing GraphQL setup:

```graphql
type Todo {
  id: String!
  title: String!
  done: Boolean!
}

type Query {
  todos: [Todo!]!
  health: HealthStatus!
}

type Mutation {
  addTodo(title: String!): Todo!
  completeTodo(id: String!): Todo!
}
```

### ID Generation

Use any simple unique ID strategy (e.g., incrementing counter, UUID, or `str(time.time())`).

### Validation

- **`addTodo`:** Reject empty or whitespace-only `title` with a GraphQL error; do not add a todo.
- **`completeTodo`:** Reject unknown `id` with a GraphQL error; do not modify state.

### File Changes

**New files:**
- `src/todos.py` — in-memory storage and todo operations
- `src/schema.py` — GraphQL schema definition and resolvers
- `tests/test_todos.py` — unit tests for todo operations

**Modified files:**
- `src/__init__.py` or equivalent entry point — register the GraphQL schema with the service

**Unchanged files:**
- `src/health.py` — leave `health()` operation unchanged
- `tests/test_health.py` — existing health check tests remain

---

## Acceptance Criteria

### AC1: Add a todo with valid title
**Given** the GraphQL service is running  
**When** `addTodo(title: "Buy milk")` is called  
**Then** it returns a todo with that title and `done` set to `false`

**Tests:**
- `tests/test_todos.py::test_add_todo_with_valid_title`

---

### AC2: Query all todos
**Given** at least one todo has been added  
**When** the `todos` query is called  
**Then** it returns a list including that todo

**Tests:**
- `tests/test_todos.py::test_query_todos_includes_added_todo`

---

### AC3: Complete a todo
**Given** a todo exists with a known `id`  
**When** `completeTodo(id: "<known-id>")` is called  
**Then** it returns the same todo with `done` set to `true`

**Tests:**
- `tests/test_todos.py::test_complete_todo_sets_done_to_true`

---

### AC4: Reject empty title
**Given** the GraphQL service is running  
**When** `addTodo(title: "")` is called  
**Then** it is rejected and no todo is added

**Tests:**
- `tests/test_todos.py::test_add_todo_rejects_empty_title`

---

### AC5: Reject unknown ID
**Given** the GraphQL service is running  
**When** `completeTodo(id: "<unknown-id>")` is called  
**Then** it is rejected

**Tests:**
- `tests/test_todos.py::test_complete_todo_rejects_unknown_id`

---

### AC6: Health check unchanged
**Given** the GraphQL service is running  
**When** the `health()` query is called  
**Then** it returns exactly `{"status": "ok"}` (unchanged behavior)

**Tests:**
- `tests/test_health.py::test_health_check_returns_ok` (existing test, must still pass)

---

## Verification

Each acceptance criterion maps to a test case:

1. Call `addTodo("Buy milk")` → verify returned todo has title "Buy milk" and `done` is `false`.
2. Add a todo, then call `todos` → verify the list includes that todo.
3. Add a todo, call `completeTodo` with its ID → verify returned todo has `done` set to `true`.
4. Call `addTodo("")` → verify error response and no todo added.
5. Call `completeTodo("<invalid-id>")` → verify error response.
6. Call `health()` → verify response is `{"status": "ok"}`.

All tests must pass before the story is considered complete.

---

## Notes

- **Storage simplicity:** A global list or dict in `src/todos.py` is sufficient; thread safety is not required unless the service is multi-threaded.
- **GraphQL framework:** Use the existing GraphQL setup in the repo; if none exists, add a minimal GraphQL library (e.g., `graphene` or `strawberry` for Python).
- **No external dependencies:** Avoid adding database clients, ORMs, or authentication middleware.

---

**End of Story**
