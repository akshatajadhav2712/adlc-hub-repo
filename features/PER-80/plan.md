# Plan — PER-80 (v2)

- made from: review feedback (architect_review)
- source: LLM planner
- complexity: simple
- needs plan review: yes
- track hint: quick

Single-repo behavioral change with a data type refinement; architect added to define the enum schema contract before implementation.

## What changed

Added architect to define the done field as an enum with values ['done', 'not done'] per reviewer feedback; requirements and story will be regenerated to align with the enum schema.

- Added: architect
- Kept (results reused): analyst

## Feedback that informed this version

- **PR inline comment** by akshatajadhav2712 on hub#27 `features/PER-80/requirements.md:67` `[hub#27:inline-4409652628]`: data type of done field should be enum with values [done, not done]
  - addressed by: architect added to define done as enum type in GraphQL schema

## Steps

1. [x] **analyst (inline_story)** — Single repo, well-defined behavior with complete AC in the ticket.
2. [ ] **architect (hld)** — GraphQL schema contract needs explicit enum definition for the done field.
   - focus: Define the done field as an enum type with values ['done', 'not done'] in the GraphQL schema; update the story's tech notes and acceptance criteria to reflect the enum instead of boolean.
3. [ ] **merge_pack (pack)** _(auto)_ — Pack per-ticket specs into merged_plan.md for review.
4. [ ] **architect_gate (review)** _(auto)_ _(rerun)_ — Human approval of specs and this plan before Dev.
5. [ ] **dev** _(auto)_ — Fixed execution step.
6. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
7. [ ] **open_pr** _(auto)_ — Fixed execution step.
8. [ ] **pr_review** _(auto)_ — Fixed execution step.
9. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **tpm_gate** — Scope is unambiguous and technical, no product sign-off needed.
- **scrum_master** — Single story in one repo, no breakdown required.
- **designer** — Implementation is straightforward from the AC and schema; no complex file plan needed.
- **test_architect** — AC already map 1:1 to test cases; no additional test design value.
- **critic** — Only requirements and HLD to cross-check; bounded enough without critic.
