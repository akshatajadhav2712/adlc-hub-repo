# Requirements — PER-78

**Ticket:** Add an in-memory GraphQL todo list  
**Type:** Story  
**Status:** Draft

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

## Data Model

A todo has:
- `id`: string (unique identifier)
- `title`: string (non-empty)
- `done`: boolean (default `false`)

---

## Acceptance Criteria

### AC1: Add a todo with valid title
**Given** the GraphQL service is running  
**When** `addTodo(title: "Buy milk")` is called  
**Then** it returns a todo with that title and `done` set to `false`

---

### AC2: Query all todos
**Given** at least one todo has been added  
**When** the `todos` query is called  
**Then** it returns a list including that todo

---

### AC3: Complete a todo
**Given** a todo exists with a known `id`  
**When** `completeTodo(id: "<known-id>")` is called  
**Then** it returns the same todo with `done` set to `true`

---

### AC4: Reject empty title
**Given** the GraphQL service is running  
**When** `addTodo(title: "")` is called  
**Then** it is rejected and no todo is added

---

### AC5: Reject unknown ID
**Given** the GraphQL service is running  
**When** `completeTodo(id: "<unknown-id>")` is called  
**Then** it is rejected

---

### AC6: Health check unchanged
**Given** the GraphQL service is running  
**When** the `health()` query is called  
**Then** it returns exactly `{"status": "ok"}` (unchanged behavior)

---

## Technical Notes

- **Storage:** Use an in-memory data structure (e.g., list or dict) to hold todos; no database.
- **GraphQL schema:** Define a `Todo` type and three operations (`todos`, `addTodo`, `completeTodo`) in the existing GraphQL setup.
- **ID generation:** Use any simple unique ID strategy (e.g., incrementing counter, UUID).
- **Validation:** Enforce non-empty title on `addTodo`; return error for unknown ID on `completeTodo`.

---

## Verification

Each acceptance criterion maps to a test case:
1. Call `addTodo("Buy milk")` → verify returned todo has title "Buy milk" and `done` is `false`.
2. Add a todo, then call `todos` → verify the list includes that todo.
3. Add a todo, call `completeTodo` with its ID → verify returned todo has `done` set to `true`.
4. Call `addTodo("")` → verify error response and no todo added.
5. Call `completeTodo("<invalid-id>")` → verify error response.
6. Call `health()` → verify response is `{"status": "ok"}`.

---

**End of Requirements**
