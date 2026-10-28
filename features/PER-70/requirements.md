# Requirements — PER-70

**Story:** Add an in-memory GraphQL todo list  
**Ticket ID:** PER-70  
**Epic:** (none)  
**Type:** Story  
**Complexity:** Simple

---

## Overview

Add three GraphQL operations to the existing `graphql` service for managing an in-memory todo list. The list is ephemeral (resets on process restart) and lives alongside the unchanged `health()` endpoint. No database, no authentication, no UI.

---

## Acceptance Criteria

### AC1: Add a todo with valid title

**Given** the todo list is empty  
**When** I call `addTodo(title: "Buy milk")`  
**Then** the operation returns a todo with:
- `id`: a non-empty string
- `title`: "Buy milk"
- `done`: false

### AC2: Query all todos

**Given** I have added a todo with title "Buy milk"  
**When** I call `todos`  
**Then** the response includes the todo from AC1

### AC3: Complete a todo

**Given** a todo exists with `id` "abc123" and `done` is false  
**When** I call `completeTodo(id: "abc123")`  
**Then** the operation returns the same todo with `done` set to true

### AC4: Reject empty title

**Given** the todo list is in any state  
**When** I call `addTodo(title: "")`  
**Then** the operation is rejected with an error  
**And** no todo is added to the list

### AC5: Reject unknown todo ID

**Given** no todo exists with `id` "unknown"  
**When** I call `completeTodo(id: "unknown")`  
**Then** the operation is rejected with an error

### AC6: Health endpoint unchanged

**Given** the service is running  
**When** I call the `health()` endpoint  
**Then** it returns exactly `{"status": "ok"}`  
**And** the response format is unchanged from the current implementation

---

## Data Model

### Todo

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `id` | String | Yes | (generated) | Unique identifier assigned at creation |
| `title` | String | Yes | (none) | Must be non-empty |
| `done` | Boolean | Yes | `false` | Completion status |

---

## Operations

### Query: `todos`

**Returns:** List of all todos in memory  
**Parameters:** None  
**Behavior:** Returns an empty list when no todos exist

### Mutation: `addTodo`

**Parameters:**
- `title: String!` — The todo title (required, must be non-empty)

**Returns:** The created todo  
**Errors:**
- Validation error when `title` is empty or whitespace-only

### Mutation: `completeTodo`

**Parameters:**
- `id: String!` — The todo ID (required)

**Returns:** The updated todo with `done` set to `true`  
**Errors:**
- Not found error when `id` does not match any existing todo

---

## Constraints

1. **In-memory only:** The todo list lives in process memory and is lost on restart.
2. **Single service:** Changes are confined to the `graphql` repository; `agent` and `mcp` remain untouched.
3. **Preserve existing behavior:** The `health()` endpoint must remain exactly as implemented.
4. **No external dependencies:** No database, no authentication, no persistent storage.

---

## Out of Scope

- Persistent storage (database, file system)
- User authentication or authorization
- Web UI or client application
- Editing todo titles after creation
- Deleting todos
- Filtering, sorting, or pagination
- Cross-service communication

---

## Verification Approach

Test each acceptance criterion with:
- Unit tests for the GraphQL resolver functions
- Integration tests exercising the GraphQL schema
- Explicit test cases for validation errors (AC4, AC5)
- Regression test confirming `health()` output unchanged (AC6)
