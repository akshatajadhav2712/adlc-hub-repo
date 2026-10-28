# Story PER-72 — Add an in-memory GraphQL todo list

**Type:** story  
**Status:** draft  
**Repo:** `graphql`

---

## Overview

Extend the existing `graphql` service with an in-memory todo list API. The `health()` query remains unchanged. No UI, authentication, database, or persistence; the list resets on process restart.

---

## Scope

**One repo only:** `graphql`

**Do not change:** `agent`, `mcp`

**No:** UI, login, database, persistence

---

## Requirements Coverage

This story implements all acceptance criteria from the ticket:

- **AC1:** `addTodo("Buy milk")` returns a todo with that title and `done = false`
- **AC2:** `todos` includes that todo
- **AC3:** `completeTodo` on that id returns the same todo with `done = true`
- **AC4:** `addTodo("")` is rejected and adds nothing
- **AC5:** `completeTodo` with an unknown id is rejected
- **AC6:** `health()` still returns exactly `{"status": "ok"}`

---

## Technical Specification

### Data Model

A **todo** has:

- `id`: string (UUID or counter-based)
- `title`: string
- `done`: boolean (default `false`)

The list is held in memory (module-level list or dictionary) and is empty when the process starts.

### GraphQL Schema

Add to the existing schema:

```graphql
type Todo {
  id: String!
  title: String!
  done: Boolean!
}

type Query {
  todos: [Todo!]!
}

type Mutation {
  addTodo(title: String!): Todo!
  completeTodo(id: String!): Todo!
}
```

The existing `health` query remains unchanged.

### Operations

#### Query: `todos`

- Returns all todos in the in-memory list
- Returns: `[Todo!]!`

#### Mutation: `addTodo(title: String!)`

- Validates: title must not be empty
- Generates unique `id` (UUID recommended)
- Creates todo with `done = false`
- Adds to in-memory list
- Returns: `Todo!` on success
- Rejects: empty title with appropriate GraphQL error

#### Mutation: `completeTodo(id: String!)`

- Validates: `id` must exist in the list
- Sets `done = true` on the matching todo
- Returns: `Todo!` on success
- Rejects: unknown `id` with appropriate GraphQL error

### Implementation Notes

- Store todos in `src/todos.py` as a module-level list or dictionary
- Use Python's `uuid` module for id generation
- GraphQL resolver functions in `src/resolvers.py` or inline in schema setup
- No changes to `src/health.py`

---

## Files Changed

### New Files

- `src/todos.py` — in-memory todo storage and business logic
- `src/schema.py` or extend existing schema file — GraphQL type definitions and resolvers
- `tests/test_todos.py` — test coverage for all six acceptance criteria

### Modified Files

- `src/__init__.py` — may need to register new resolvers or schema extensions

### Unchanged Files

- `src/health.py` — no changes
- `tests/test_health.py` — no changes

---

## Acceptance Criteria

### AC1: Add a todo with a valid title

**Given** the todo list is empty  
**When** `addTodo("Buy milk")` is called  
**Then** the mutation returns a todo with `title = "Buy milk"` and `done = false`

**Verification:**  
GraphQL mutation test in `tests/test_todos.py` confirms returned todo shape and values.

---

### AC2: Query returns all todos

**Given** a todo was added with `addTodo("Buy milk")`  
**When** `todos` is queried  
**Then** the result includes that todo

**Verification:**  
GraphQL query test in `tests/test_todos.py` confirms the added todo appears in the list.

---

### AC3: Complete a todo by id

**Given** a todo was added with `addTodo("Buy milk")` and returned `id = "xyz"`  
**When** `completeTodo("xyz")` is called  
**Then** the mutation returns the same todo with `done = true`

**Verification:**  
GraphQL mutation test in `tests/test_todos.py` confirms `done` field is `true` after completion.

---

### AC4: Reject empty title

**Given** the todo list is in any state  
**When** `addTodo("")` is called  
**Then** the mutation is rejected and no todo is added

**Verification:**  
GraphQL mutation test in `tests/test_todos.py` confirms error response and list count unchanged.

---

### AC5: Reject unknown id

**Given** no todo exists with `id = "unknown"`  
**When** `completeTodo("unknown")` is called  
**Then** the mutation is rejected

**Verification:**  
GraphQL mutation test in `tests/test_todos.py` confirms error response.

---

### AC6: Health query unchanged

**Given** the service is running  
**When** `health()` is queried  
**Then** it returns exactly `{"status": "ok"}`

**Verification:**  
Existing test in `tests/test_health.py` still passes; no changes to health resolver.

---

## Tests

All six acceptance criteria are verified by GraphQL integration tests in `tests/test_todos.py`:

1. `test_add_todo_with_valid_title` — AC1
2. `test_query_todos_includes_added_todo` — AC2
3. `test_complete_todo_by_id` — AC3
4. `test_add_todo_rejects_empty_title` — AC4
5. `test_complete_todo_rejects_unknown_id` — AC5
6. `test_health_unchanged` — AC6 (or confirm existing `test_health.py` still passes)

---

## Out of Scope

- Persistence (file, database)
- Authentication or authorization
- UI or frontend
- Cross-service integration
- Todo deletion, editing, filtering, or sorting beyond the specified operations

---

## Notes

- In-memory storage means todos are lost on process restart (by design)
- `id` generation strategy: UUID recommended for uniqueness
- GraphQL framework and server setup assumed to already exist in `graphql` repo
- No schema migration or database setup needed

---

**End of Story**
