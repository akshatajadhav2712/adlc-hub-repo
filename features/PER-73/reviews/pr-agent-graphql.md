```json
[
  {
    "path": "src/resolvers.py",
    "line": 104,
    "body": "**AC5 violation:** `addReminder` mutation does not accept `description` parameter in resolver signature, but the GraphQL schema (line 46 of `src/schema.py`) and LLD (Story 2, Resolver Signatures section) both declare `description: String` as an optional argument. The resolver must accept `description: str | None = None` to match the schema contract."
  },
  {
    "path": "src/resolvers.py",
    "line": 110,
    "body": "**AC5 violation:** `add_reminder` call omits the `description` argument when forwarding to the store layer. The LLD specifies `add_reminder(listId, text, dueAt, description)` in `src/store.py` line 91–96, and the store implementation at line 91 of the diff correctly accepts `description`. The resolver must pass `description` through: `add_reminder(listId, text, dueAt, description)`."
  }
]
```
