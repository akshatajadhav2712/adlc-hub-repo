# Plan — PER-73 (v1)

- made from: ticket
- source: LLM planner
- complexity: moderate
- needs plan review: yes
- track hint: standard

This feature adds two-part in-memory reminder functionality to the GraphQL service with clear data models, operations, and validation rules; the structured nature and product decisions call for requirements analysis and breakdown, while the single-repo scope keeps the plan lightweight.

## Steps

1. [x] **analyst (requirements)** — The ticket contains product decisions awaiting review and multiple acceptance criteria that need formal Given/When/Then requirements.
   - focus: Focus on the product decisions (uniqueness, cascading deletes, validation limits); map each AC to requirements; ensure Part 1/Part 2 ordering is clear.
2. [ ] **tpm_gate (review)** — Five product decisions are explicitly flagged for review before technical work begins.
3. [ ] **ticketize (normalize)** _(auto)_ — Normalize proposed tickets after TPM approval.
4. [ ] **scrum_master (breakdown)** — The work splits into Part 1 (lists) and Part 2 (reminders) with a dependency, requiring ordered breakdown into implementable stories.
   - focus: Create one story for reminder lists (Part 1) and one for reminders (Part 2); ensure Part 2 depends on Part 1; each story covers its mutations, queries, validation, and tests.
5. [ ] **designer (lld)** — Each story needs concrete implementation details: in-memory data structure choices, GraphQL schema definitions, and validation logic.
   - focus: Specify the in-memory store design (dictionaries vs classes), GraphQL type definitions, resolver signatures, and validation flows for each operation.
6. [ ] **test_architect (tests)** — Nine acceptance criteria with edge cases (duplicates, blanks, limits, past dates) need explicit test cases mapped 1:1.
   - focus: Map each AC to test cases; cover positive paths, validation rejections, and edge cases like blank strings and boundary limits.
7. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
8. [ ] **dev** _(auto)_ — Fixed execution step.
9. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
10. [ ] **open_pr** _(auto)_ — Fixed execution step.
11. [ ] **pr_review** _(auto)_ — Fixed execution step.
12. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **architect** — Single repo with in-memory storage, no cross-service contracts or persistent data; no architectural decisions needed beyond what the LLD covers.
- **critic** — With analyst requirements, breakdown, and designer LLD in sequence, the cross-check adds minimal value; test_architect already ensures AC coverage.
