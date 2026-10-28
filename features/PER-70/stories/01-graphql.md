# Story — Add an in-memory GraphQL todo list

**ID:** PER-70  
**Type:** Story  
**Status:** Draft  
**Complexity:** Simple  
**Epic:** (none)

---

## Description

Add three GraphQL operations to the existing `graphql` service for managing an in-memory todo list. The list is ephemeral (resets on process restart) and lives alongside the unchanged `health()` endpoint. No database, no authentication, no UI.

**Scope:** Changes are confined to the `graphql` repository. The `agent` and `mcp` repositories remain untouched.

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

## Technical Specification

### Data Model

#### Todo

| Field | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `id` | String | Yes | (generated) | Unique identifier assigned at creation |
| `title` | String | Yes | (none) | Must be non-empty |
| `done` | Boolean | Yes | `false` | Completion status |

### GraphQL Schema

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

### Operations

#### Query: `todos`

- **Returns:** List of all todos in memory
- **Parameters:** None
- **Behavior:** Returns an empty list when no todos exist

#### Mutation: `addTodo`

- **Parameters:**
  - `title: String!` — The todo title (required, must be non-empty)
- **Returns:** The created todo
- **Validation:**
  - Reject when `title` is empty string or whitespace-only
  - Error message should indicate invalid input
- **ID Generation:** Use UUID or timestamp-based unique string

#### Mutation: `completeTodo`

- **Parameters:**
  - `id: String!` — The todo ID (required)
- **Returns:** The updated todo with `done` set to `true`
- **Validation:**
  - Reject when `id` does not match any existing todo
  - Error message should indicate todo not found

### Implementation Approach

#### File Structure

```
src/
  __init__.py          (existing)
  health.py            (existing, unchanged)
  todo_store.py        (new: in-memory storage)
  todo_schema.py       (new: GraphQL schema definition)
  todo_resolvers.py    (new: resolver functions)

tests/
  __init__.py          (existing)
  test_health.py       (existing, unchanged)
  test_todo_store.py   (new: store unit tests)
  test_todo_schema.py  (new: GraphQL integration tests)
```

#### In-Memory Storage (`src/todo_store.py`)

- Implement a simple Python class managing a list or dict of todos
- Methods: `add_todo(title)`, `get_all_todos()`, `complete_todo(id)`, `find_by_id(id)`
- ID generation using `uuid.uuid4().hex` or similar
- Validation logic for empty titles and unknown IDs
- Thread-safety not required (single-process assumption)

#### GraphQL Schema (`src/todo_schema.py`)

- Define the schema string with Todo type, Query, and Mutation
- Use `strawberry`, `ariadne`, or `graphene` (match existing project conventions)
- If no GraphQL library is present, choose `strawberry` for simplicity

#### Resolvers (`src/todo_resolvers.py`)

- Implement resolver functions for `todos`, `addTodo`, and `completeTodo`
- Delegate to `todo_store` for business logic
- Translate store errors (e.g., "not found") into GraphQL errors
- Preserve existing `health()` resolver unchanged

### Constraints

1. **In-memory only:** The todo list lives in process memory and is lost on restart
2. **Single service:** Changes are confined to the `graphql` repository
3. **Preserve existing behavior:** The `health()` endpoint in `src/health.py` must remain exactly as implemented
4. **No external dependencies:** No database, no authentication, no persistent storage
5. **Validation is strict:**
   - Empty string title: always rejected
   - Whitespace-only title: always rejected
   - Unknown ID: always rejected

### Out of Scope

- Persistent storage (database, file system)
- User authentication or authorization
- Web UI or client application
- Editing todo titles after creation
- Deleting todos
- Filtering, sorting, or pagination
- Cross-service communication

---

## Verification

### Unit Tests (`tests/test_todo_store.py`)

1. **Test add_todo with valid title**
   - Call `add_todo("Buy milk")`
   - Assert returned todo has `id`, `title="Buy milk"`, `done=False`

2. **Test add_todo rejects empty title**
   - Call `add_todo("")`
   - Assert raises validation error

3. **Test get_all_todos returns added todo**
   - Add "Buy milk"
   - Call `get_all_todos()`
   - Assert list contains the added todo

4. **Test complete_todo with valid id**
   - Add todo, capture its id
   - Call `complete_todo(id)`
   - Assert returned todo has `done=True`

5. **Test complete_todo rejects unknown id**
   - Call `complete_todo("unknown")`
   - Assert raises not-found error

### Integration Tests (`tests/test_todo_schema.py`)

1. **AC1: Add a todo with valid title**
   - Execute `addTodo(title: "Buy milk")` mutation
   - Assert response contains `id`, `title`, `done=False`

2. **AC2: Query all todos**
   - Add "Buy milk"
   - Execute `todos` query
   - Assert response includes the added todo

3. **AC3: Complete a todo**
   - Add todo, capture its id
   - Execute `completeTodo(id: "<id>")` mutation
   - Assert response has `done=True`

4. **AC4: Reject empty title**
   - Execute `addTodo(title: "")` mutation
   - Assert returns error
   - Execute `todos` query
   - Assert list is empty

5. **AC5: Reject unknown todo ID**
   - Execute `completeTodo(id: "unknown")` mutation
   - Assert returns error

### Regression Tests (`tests/test_health.py`)

6. **AC6: Health endpoint unchanged**
   - Call `health()` endpoint
   - Assert returns exactly `{"status": "ok"}`
   - Verify no additional fields or format changes

---

## Notes

- **Focus:** The three GraphQL operations and their exact validation rules; preserve the existing `health()` endpoint unchanged.
- **GraphQL library:** If not already present in the project, add `strawberry-graphql` as a dependency for Python GraphQL support.
- **ID uniqueness:** Use Python's `uuid` module for reliable unique IDs.
- **Error messages:** GraphQL errors should clearly distinguish between validation failures (empty title) and not-found errors (unknown ID).

---

## References

- Ticket: PER-70
- Requirements: `requirements.md` (same acceptance criteria)
- Repository: `graphql` (`/home/aksj/Documents/adlc-graphql`)
