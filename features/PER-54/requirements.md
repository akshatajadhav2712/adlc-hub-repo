# requirements.md

**Story: In-memory GraphQL todo list**  
**ID:** PER-54.1  
**Epic:** PER-54  
**Type:** story  
**Repository:** graphql

---

## Overview

Add three GraphQL operations (queries and mutations) to the graphql service for managing an in-memory todo list. Each todo has an id, title, and done flag. The list starts empty on process start and is not persisted.

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

## Data Model

Each todo object has:

- **id**: string, unique identifier assigned at creation
- **title**: string, the todo description
- **done**: boolean, defaults to `false` at creation

---

## Operations

### Query: todos

Returns the full list of todos currently in memory.

**Returns:** list of todo objects

---

### Mutation: addTodo(title: String!)

Creates a new todo with the given title and `done = false`.

**Arguments:**
- `title` (required string)

**Returns:** the created todo object

**Validation:** rejects empty title

---

### Mutation: completeTodo(id: String!)

Sets the `done` field to `true` for the todo with the given id.

**Arguments:**
- `id` (required string)

**Returns:** the updated todo object

**Validation:** rejects unknown id

---

## Constraints

- No database or persistent storage; the list lives in memory only
- The list is empty when the process starts
- Changes to other operations (e.g., `health()`) are out of scope
- Changes to the `agent` or `mcp` repositories are out of scope
- No authentication, authorization, or user interface

---

## Success Signal

All six acceptance criteria pass as automated tests.
