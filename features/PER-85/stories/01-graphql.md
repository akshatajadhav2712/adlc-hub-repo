# Story — PER-85: Add an in-memory GraphQL todo list

## Metadata

```yaml
id: PER-85
type: story
title: Add an in-memory GraphQL todo list
epic: (none)
status: draft
assignee: (unassigned)
created: 2025-01-22
updated: 2025-01-22
repository: graphql
complexity: simple
track: quick
```

## Overview

Extend the existing `graphql` service with an in-memory todo list. The service already exposes a `health()` query that returns `{"status": "ok"}`; that query remains unchanged. The new todo list lives in memory only and is reset when the process restarts.

## Scope

**In scope:**
- Add an in-memory todo list to the `graphql` repository at `/home/aksj/Documents/adlc-graphql`
- Three new GraphQL operations: `todos`, `addTodo`, `completeTodo`
- Data model: todo with `id` (string), `title` (string), `done` (boolean, default false)

**Out of scope:**
- Changes to `agent` or `mcp` repositories
- UI, authentication, or authorization
- Persistent storage (database, file system, etc.)
- Modification of the existing `health()` query

## Technical Specification

### Data Model

A todo item has three fields:

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | string | (generated) | Unique identifier |
| `title` | string | (required) | Todo description |
| `done` | boolean | `false` | Completion status |

### Operations

#### Query: `todos`

**Returns:** List of all todos in memory

**Behavior:** Returns every todo currently stored in the in-memory list.

#### Mutation: `addTodo(title: String!)`

**Input:** `title` (required, non-empty string)

**Returns:** The newly created todo

**Behavior:**
- Creates a new todo with a unique `id`, the provided `title`, and `done` set to `false`
- Adds the todo to the in-memory list
- Returns the created todo

**Validation:**
- Empty string (`""`) is rejected
- No todo is added when validation fails

#### Mutation: `completeTodo(id: String!)`

**Input:** `id` (required, string)

**Returns:** The updated todo

**Behavior:**
- Finds the todo with the given `id`
- Sets its `done` field to `true`
- Returns the updated todo

**Validation:**
- Unknown `id` is rejected
- No todo is modified when validation fails

### Implementation Approach

**Files to create:**
- `src/todo.py` — In-memory todo list with operations (storage, add, complete)
- `tests/test_todo.py` — Test coverage for all acceptance criteria

**Files unchanged:**
- `src/health.py` — Existing health check remains exactly as is
- `tests/test_health.py` — Existing health tests remain unchanged

**Implementation notes:**
- Use a simple in-memory list or dictionary to store todos
- Generate unique `id` values (e.g., UUID or incremental counter)
- Validate inputs before mutation operations
- Return appropriate errors for invalid operations

## Acceptance Criteria

### AC1: Add a todo with a valid title
**Given** the in-memory todo list is empty  
**When** `addTodo("Buy milk")` is called  
**Then** a todo is returned with `title` "Buy milk" and `done` `false`

**Verify:** Call `addTodo("Buy milk")` and assert the returned todo has `title == "Buy milk"` and `done == False`.

### AC2: Query returns added todos
**Given** a todo "Buy milk" has been added  
**When** `todos` query is executed  
**Then** the response includes the "Buy milk" todo

**Verify:** Add a todo via `addTodo("Buy milk")`, then call `todos` and assert the list contains the added todo.

### AC3: Complete a todo by id
**Given** a todo with a known `id` exists  
**When** `completeTodo` is called with that `id`  
**Then** the same todo is returned with `done` set to `true`

**Verify:** Add a todo, capture its `id`, call `completeTodo(id)`, and assert the returned todo has `done == True`.

### AC4: Reject empty title
**Given** the in-memory todo list  
**When** `addTodo("")` is called  
**Then** the operation is rejected and no todo is added to the list

**Verify:** Call `addTodo("")` and assert it raises an error or returns an error response. Verify the todo list count remains unchanged.

### AC5: Reject unknown id for completion
**Given** the in-memory todo list  
**When** `completeTodo` is called with an unknown `id`  
**Then** the operation is rejected and no todo is modified

**Verify:** Call `completeTodo("nonexistent-id")` and assert it raises an error or returns an error response. Verify no todos are modified.

### AC6: Health query remains unchanged
**Given** the service is running  
**When** `health()` query is executed  
**Then** it returns exactly `{"status": "ok"}` (unchanged behavior)

**Verify:** Call `health()` and assert the response is exactly `{"status": "ok"}`. Run existing `tests/test_health.py` and verify all tests pass.

## Notes

- The in-memory list resets when the process restarts
- No frameworks, ORMs, databases, or cloud stacks are prescribed
- The `graphql` repository already has a working `health()` query at `src/health.py`; preserve it exactly
- All six acceptance criteria from the ticket are preserved verbatim

## References

- Ticket: PER-85
- Repository: `graphql` at `/home/aksj/Documents/adlc-graphql`
- Existing files: `src/health.py`, `tests/test_health.py`
