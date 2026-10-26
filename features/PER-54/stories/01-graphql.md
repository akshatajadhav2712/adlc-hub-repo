# Story: In-memory GraphQL todo list

**ID:** PER-54.1  
**Epic:** PER-54  
**Type:** story  
**Repository:** graphql  
**Complexity:** simple  
**Track:** quick

---

## Overview

Add three GraphQL operations (queries and mutations) to the graphql service for managing an in-memory todo list. Each todo has an id, title, and done flag. The list starts empty on process start and is not persisted. The existing `health()` query remains unchanged.

---

## Acceptance Criteria

### AC1: addTodo with valid title creates and returns a todo

**Given** the service is running and the todo list is empty  
**When** a client calls `addTodo(title: "Buy milk")`  
**Then** the mutation returns a todo with `title = "Buy milk"`, `done = false`, and a non-empty `id`

---

### AC2: todos query returns all added todos

**Given** a todo with title "Buy milk" has been added  
**When** a client calls the `todos` query  
**Then** the response includes that todo with its `id`, `title = "Buy milk"`, and `done = false`

---

### AC3: completeTodo marks a todo as done

**Given** a todo with title "Buy milk" exists with `done = false`  
**When** a client calls `completeTodo(id: "<that todo's id>")`  
**Then** the mutation returns the same todo with `done = true`

---

### AC4: addTodo with empty title is rejected

**Given** the service is running  
**When** a client calls `addTodo(title: "")`  
**Then** the mutation returns a GraphQL error and does not create a todo

---

### AC5: completeTodo with unknown id is rejected

**Given** no todo exists with id "nonexistent"  
**When** a client calls `completeTodo(id: "nonexistent")`  
**Then** the mutation returns a GraphQL error

---

### AC6: health endpoint remains unchanged

**Given** the service is running with the new todo operations  
**When** a client queries `health()`  
**Then** the response is exactly `{"status": "ok"}` as before

---

## Technical Specification

### Data Model

Each todo object has three fields:

- **id**: string, unique identifier assigned at creation (use UUID or incrementing counter)
- **title**: string, the todo description
- **done**: boolean, defaults to `false` at creation

### In-Memory Storage

Maintain a module-level list or dictionary in a new file `src/todos.py`:

```python
# src/todos.py
todos_store = []  # or {} if using dict for id lookup
```

### GraphQL Schema

Extend the existing schema (or create `src/schema.py` if none exists) with:

```graphql
type Todo {
  id: String!
  title: String!
  done: Boolean!
}

type Query {
  health: HealthResponse!
  todos: [Todo!]!
}

type Mutation {
  addTodo(title: String!): Todo!
  completeTodo(id: String!): Todo!
}
```

### Resolvers

Create `src/resolvers.py` or add to existing resolver module:

#### Query: `todos`

- Returns the full list from `todos_store`
- No arguments
- Returns empty list `[]` on fresh start

#### Mutation: `addTodo(title: String!)`

- **Validation:** Reject if `title` is empty string; raise GraphQL error with message "Title cannot be empty"
- Generate unique `id` (UUID recommended)
- Create todo object: `{"id": <id>, "title": <title>, "done": False}`
- Append to `todos_store`
- Return the created todo

#### Mutation: `completeTodo(id: String!)`

- **Validation:** Search `todos_store` for matching `id`; if not found, raise GraphQL error with message "Todo not found"
- Set `done = True` on the matched todo
- Return the updated todo

### File Changes

| File | Change |
|------|--------|
| `src/todos.py` | Create new file with in-memory storage and todo management functions |
| `src/schema.py` | Create or extend with Todo type, todos query, addTodo and completeTodo mutations |
| `src/resolvers.py` | Create or extend with resolver functions for todos, addTodo, completeTodo |
| `src/health.py` | No changes (verify health resolver still works) |
| `tests/test_todos.py` | Create new test file covering all six acceptance criteria |
| `tests/test_health.py` | No changes (existing health tests must still pass) |

### Testing

Create `tests/test_todos.py` with test cases mapping to each AC:

- `test_add_todo_with_valid_title` → AC1
- `test_todos_query_returns_all_todos` → AC2
- `test_complete_todo_marks_as_done` → AC3
- `test_add_todo_with_empty_title_rejected` → AC4
- `test_complete_todo_with_unknown_id_rejected` → AC5
- `test_health_unchanged` → AC6 (can be in existing `test_health.py`)

---

## Constraints

- **Scope:** graphql repository only
- **No persistence:** The list is emptied on process restart
- **No UI:** Operations are GraphQL-only
- **No authentication:** Open access to all operations
- **No changes to:** `src/health.py` logic (only verify it still works)
- **Out of scope:** agent repo, mcp repo, database integration, user accounts

---

## Success Signal

All six acceptance criteria pass as automated tests. The GraphQL service responds to `todos`, `addTodo`, and `completeTodo` operations while `health()` continues to return `{"status": "ok"}`.

---

## Notes

- **ID generation:** Use Python's `uuid.uuid4()` for unique todo IDs, or a simple counter if UUIDs are overkill
- **Error handling:** GraphQL errors should use appropriate error types (e.g., `ValueError` or custom GraphQL exception)
- **Test isolation:** Each test should start with an empty `todos_store` or reset it in setup
- **Validation order:** Check title emptiness before creating the todo object
