# Requirements — PER-76

**Ticket:** Add an in-memory GraphQL todo list  
**Type:** Story  
**Epic:** (none)

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

## Technical Notes

- **Schema:** Define a `Todo` type with fields `id: String!`, `title: String!`, `done: Boolean!`.
- **Resolvers:** Implement query `todos: [Todo!]!` and mutations `addTodo(title: String!): Todo!`, `completeTodo(id: String!): Todo!`.
- **Storage:** Use an in-memory data structure (e.g., dict or list) to hold todos; no persistence layer.
- **ID generation:** Use any approach that guarantees unique string IDs per todo (e.g., UUID, counter).
- **Validation:** `addTodo` must reject empty strings; `completeTodo` must reject unknown IDs (both should raise a GraphQL error).
- **Existing health:** Do not modify `src/health.py` or its behavior.

---

## Out of Scope

- Persistent storage (database, file)
- Authentication or authorization
- Frontend UI
- Editing or deleting todos
- Changes to the `agent` or `mcp` repos

---

## Dependencies

None.

---

## Open Questions

None.
