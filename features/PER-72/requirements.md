# Requirements — PER-72

**Add an in-memory GraphQL todo list**

---

## Overview

Extend the existing `graphql` service with an in-memory todo list API. The `health()` query remains unchanged. No UI, authentication, database, or persistence; the list resets on process restart.

---

## Scope

**One repo only:** `graphql`

**Do not change:** `agent`, `mcp`

**No:** UI, login, database, persistence

---

## Data Model

A **todo** has:

- `id`: string
- `title`: string
- `done`: boolean (default `false`)

The list is held in memory and is empty when the process starts.

---

## Operations

### Query: `todos`

Returns all todos in the list.

**Returns:** `[Todo!]!`

---

### Mutation: `addTodo(title: String!)`

Creates a new todo with the given title and `done = false`.

**Returns:** `Todo!` on success

**Rejects:** empty title (`title = ""`)

---

### Mutation: `completeTodo(id: String!)`

Sets `done = true` on the todo with the given `id`.

**Returns:** `Todo!` on success

**Rejects:** unknown `id`

---

## Acceptance Criteria

### AC1: Add a todo with a valid title

**Given** the todo list is empty  
**When** `addTodo("Buy milk")` is called  
**Then** the mutation returns a todo with `title = "Buy milk"` and `done = false`

---

### AC2: Query returns all todos

**Given** a todo was added with `addTodo("Buy milk")`  
**When** `todos` is queried  
**Then** the result includes that todo

---

### AC3: Complete a todo by id

**Given** a todo was added with `addTodo("Buy milk")` and returned `id = "xyz"`  
**When** `completeTodo("xyz")` is called  
**Then** the mutation returns the same todo with `done = true`

---

### AC4: Reject empty title

**Given** the todo list is in any state  
**When** `addTodo("")` is called  
**Then** the mutation is rejected and no todo is added

---

### AC5: Reject unknown id

**Given** no todo exists with `id = "unknown"`  
**When** `completeTodo("unknown")` is called  
**Then** the mutation is rejected

---

### AC6: Health query unchanged

**Given** the service is running  
**When** `health()` is queried  
**Then** it returns exactly `{"status": "ok"}`

---

## Technical Notes

- Store todos in a module-level list or dictionary in memory
- Generate unique `id` values (e.g., UUID, counter)
- GraphQL schema must define `Todo` type and the three operations
- `health()` resolver is unchanged

---

## Out of Scope

- Persistence (file, database)
- Authentication or authorization
- UI or frontend
- Cross-service integration
- Todo deletion, editing, filtering, or sorting beyond the specified operations

---

**End of Requirements**
