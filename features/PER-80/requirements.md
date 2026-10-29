# Requirements — Add an in-memory GraphQL todo list

**Story ID:** PER-80  
**Epic:** N/A (standalone story)  
**Repository:** graphql  
**Scope:** Add in-memory todo list operations to the existing graphql service without modifying health()

---

## Overview

The graphql service currently exposes a `health()` query. This story adds a small in-memory todo list feature to the same repository. The todo list lives in memory only and resets when the process restarts. No database, authentication, or UI is required.

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

## Data Model

A **todo** consists of:
- `id`: string (unique identifier)
- `title`: string (non-empty)
- `done`: boolean (default: false)

The todo list is stored **in memory only**. When the process restarts, the list is empty.

---

## GraphQL Operations

### Query: `todos`
- **Returns:** List of all todos
- **Parameters:** None

### Mutation: `addTodo`
- **Parameters:** `title: String!`
- **Returns:** The newly created todo
- **Validation:** Title must not be empty

### Mutation: `completeTodo`
- **Parameters:** `id: String!`
- **Returns:** The updated todo with `done` = true
- **Validation:** ID must exist in the list

---

## Constraints

1. **Single repository only:** All changes must be made in the `graphql` repository. No changes to `agent` or `mcp` repositories.
2. **In-memory storage:** The todo list must be stored in memory. No database, file system, or persistent storage.
3. **No UI:** This is a backend-only feature. No web interface is required.
4. **No authentication:** No login or user management is required.
5. **Health endpoint preservation:** The existing `health()` query must remain functionally unchanged and return exactly `{"status": "ok"}`.
6. **Process restart behavior:** The todo list must be empty when the service process restarts.

---

## Non-Goals

- Persistent storage (database, files)
- User authentication or authorization
- Web UI or frontend
- Cross-repository changes
- Advanced todo features (due dates, priorities, categories, etc.)
- Todo editing (changing title after creation)
- Todo deletion

---

## Success Signal

The story is complete when:
1. All six acceptance criteria pass
2. The `health()` query continues to return exactly `{"status": "ok"}`
3. The todo list behavior is verified through tests
4. The implementation is contained entirely within the `graphql` repository

---

## Verification Approach

Each acceptance criterion maps to one or more test cases:
- **AC1:** Test `addTodo` with valid title returns correct object structure
- **AC2:** Test `todos` query returns previously added todo
- **AC3:** Test `completeTodo` updates `done` flag correctly
- **AC4:** Test `addTodo` with empty string is rejected
- **AC5:** Test `completeTodo` with nonexistent ID is rejected
- **AC6:** Test `health` query output is unchanged
