---
key: PER-78
type: story
status: todo
blocks: []
blocked_by: []
labels: []
repos: []
summary: Add an in-memory GraphQL todo list
---

The graphql service already exposes health(). Add a small in-memory todo list in that same repo. Leave health() unchanged.

One repo only: graphql. Do not change agent or mcp. No UI, no login, no database. The list lives in memory and is empty again when the process restarts.

A todo has:

* id: string
* title: string
* done: boolean, default false

GraphQL operations:

* todos: return every todo
* addTodo(title: String!): create a todo and return it
* completeTodo(id: String!): set done to true and return it

Acceptance:

* addTodo("Buy milk") returns a todo with that title and done false.
* todos includes that todo.
* completeTodo on that id returns the same todo with done true.
* addTodo("") is rejected and adds nothing.
* completeTodo with an unknown id is rejected.
* health() still returns exactly {"status": "ok"}.
