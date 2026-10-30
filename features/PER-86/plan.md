# Plan — PER-86 (v1)

- made from: ticket
- source: LLM planner
- complexity: trivial
- needs plan review: yes
- track hint: quick

Self-contained utility function with explicit acceptance criteria and file locations; analyst inline_story alone suffices.

## Steps

1. [ ] **analyst (inline_story)** — Ticket has clear requirements, file paths, and test coverage expectations.
   - focus: AC and tech notes must reflect the VERSION file location, the fallback logic, and the test isolation requirement (temp dir).
2. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
3. [ ] **dev** _(auto)_ — Fixed execution step.
4. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
5. [ ] **open_pr** _(auto)_ — Fixed execution step.
6. [ ] **pr_review** _(auto)_ — Fixed execution step.
7. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **tpm_gate** — Scope is clear and internal; no product sign-off needed.
- **architect** — No new components, contracts, or cross-repo concerns; single utility function.
- **scrum_master** — Work fits in one story in one repo; no breakdown needed.
- **designer** — Implementation is straightforward from the ticket; no file plan needed.
- **test_architect** — Test cases are already explicit in AC4; no additional test design required.
- **critic** — Only one spec document; nothing to cross-check.
