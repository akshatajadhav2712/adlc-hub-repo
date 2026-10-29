# Story — Add an in-memory GraphQL todo list

**Story ID:** PER-80  
**Type:** Story  
**Epic:** N/A (standalone)  
**Repository:** `graphql`  
**Estimated Effort:** 3 points  

---

## Overview

Add a small in-memory todo list feature to the existing `graphql` service. The service already exposes `health()` and this story adds todo operations alongside it without modifying the health endpoint. The todo list lives in memory only and resets when the process restarts. No database, authentication, or UI is required.

---

## Acceptance Criteria

### AC1: Create a todo with a valid title
**Given** the graphql service is running  
**When** I execute `addTodo(title: "Buy milk")`  
**Then** the operation returns a todo object with:
- `id` (non-empty string)
- `title` = "Buy milk"
- `done` = false

### AC2: Query all todos
**Given** I have added a todo with title "Buy milk"  
**When** I execute the `todos` query  
**Then** the response includes the todo with:
- The same `id` returned from `addTodo`
- `title` = "Buy milk"
- `done` = false

### AC3: Complete a todo by ID
**Given** I have added a todo with title "Buy milk"  
**When** I execute `completeTodo(id: "<the-todo-id>")`  
**Then** the operation returns the same todo with:
- Same `id`
- Same `title`
- `done` = true

### AC4: Reject empty title
**Given** the graphql service is running  
**When** I execute `addTodo(title: "")`  
**Then** the operation is rejected with an error  
**And** no new todo is added to the list

### AC5: Reject completeTodo with unknown ID
**Given** the graphql service is running  
**When** I execute `completeTodo(id: "nonexistent-id")`  
**Then** the operation is rejected with an error  
**And** no todos are modified

### AC6: Health endpoint unchanged
**Given** the graphql service is running  
**When** I execute the `health` query  
**Then** the response is exactly `{"status": "ok"}`  
**And** the health endpoint behavior is identical to before this story

---

## Technical Specification

### Repository Scope
All changes are confined to the `graphql` repository. No modifications to `agent` or `mcp` repositories.

### Data Model
A todo consists of:
- `id`: string (unique identifier, generated on creation)
- `title`: string (non-empty)
- `done`: boolean (default: false)

### Storage
The todo list is stored **in memory only** as a module-level data structure. When the process restarts, the list is empty.

### GraphQL Schema Extensions

```graphql
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

### File Structure

#### New Files
- `src/todo.py` — Todo data model, in-memory storage, and business logic
- `src/schema.py` — GraphQL schema definition combining health and todo types
- `src/resolvers.py` — Query and mutation resolvers for todo operations
- `tests/test_todo.py` — Unit tests for todo business logic
- `tests/test_resolvers.py` — Integration tests for GraphQL resolvers

#### Modified Files
- `src/health.py` — No changes (preserved as-is)
- `tests/test_health.py` — No changes (preserved as-is)

### Implementation Details

**`src/todo.py`**
- Defines `Todo` dataclass with `id`, `title`, `done` fields
- Maintains an in-memory list: `_todos: List[Todo] = []`
- Provides functions:
  - `add_todo(title: str) -> Todo` — validates non-empty title, generates unique ID, appends to list
  - `get_all_todos() -> List[Todo]` — returns copy of todo list
  - `complete_todo(id: str) -> Todo` — finds todo by ID, sets `done=True`, raises error if not found

**`src/schema.py`**
- Defines GraphQL schema extending existing health query with todos query and mutations
- Uses Strawberry GraphQL or similar library for schema definition

**`src/resolvers.py`**
- Implements query resolver for `todos` → calls `get_all_todos()`
- Implements mutation resolver for `addTodo` → calls `add_todo(title)`, handles validation errors
- Implements mutation resolver for `completeTodo` → calls `complete_todo(id)`, handles not-found errors
- Preserves existing `health` resolver unchanged

**Error Handling**
- `addTodo("")` raises `ValueError("Title cannot be empty")`
- `completeTodo("nonexistent-id")` raises `ValueError("Todo not found")`
- GraphQL layer translates exceptions into user errors

### Verification

**Unit Tests (`tests/test_todo.py`)**
- Test `add_todo()` with valid title returns correct structure
- Test `add_todo("")` raises ValueError
- Test `complete_todo()` with valid ID sets done=True
- Test `complete_todo()` with invalid ID raises ValueError
- Test `get_all_todos()` returns all added todos

**Integration Tests (`tests/test_resolvers.py`)**
- Test GraphQL `addTodo` mutation with valid input (AC1)
- Test GraphQL `todos` query returns added todo (AC2)
- Test GraphQL `completeTodo` mutation updates todo (AC3)
- Test GraphQL `addTodo` with empty string is rejected (AC4)
- Test GraphQL `completeTodo` with unknown ID is rejected (AC5)
- Test GraphQL `health` query output unchanged (AC6)

---

## Constraints

1. **Single repository:** All changes in `graphql` repository only
2. **In-memory storage:** No database, file system, or persistent storage
3. **Health endpoint preservation:** `health()` must remain functionally unchanged and return exactly `{"status": "ok"}`
4. **No external dependencies:** Use only existing GraphQL library already in the project
5. **Process restart behavior:** Todo list must be empty on service restart

---

## Notes

- ID generation should use a simple approach (e.g., UUID or incremental counter)
- The in-memory list is not thread-safe; this is acceptable for the demo scope
- No todo deletion or editing is implemented in this story
- The `health.py` module and its tests remain completely untouched

---

## Definition of Done

- [ ] All six acceptance criteria pass
- [ ] Unit tests in `tests/test_todo.py` pass
- [ ] Integration tests in `tests/test_resolvers.py` pass
- [ ] `health()` query verified unchanged
- [ ] Code follows existing repository conventions
- [ ] No changes outside `graphql` repository
- [ ] PR approved and merged
