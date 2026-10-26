# Plan — PER-47

- source: LLM planner
- complexity: trivial
- track hint: quick

User-facing wording decision requires product sign-off before implementing a single function in one repo.

## Steps

1. [x] **analyst (requirements)** — Convert the ticket into Given/When/Then requirements that clearly distinguish the product decision from the implementation.
   - focus: Separate the product decision (choosing 'ready' vs 'available') from the implementation requirement (new function in src/health.py). Make clear that acceptance criteria are incomplete until product chooses the word.
2. [ ] **tpm_gate (review)** — Product must choose between 'ready' and 'available' before any technical work can proceed.
   - focus: This gate exists solely for product to select the readiness word. Once chosen, the implementation is trivial.
3. [ ] **ticketize (normalize)** _(auto)_ — Normalize proposed tickets after TPM approval.
4. [ ] **scrum_master (breakdown)** — Transform the requirements (with product-approved word) into one implementable story with complete AC.
   - focus: Create single story for graphql repo with the product-chosen word substituted into acceptance criteria. Include tech notes specifying src/health.py and that health() remains unchanged.
5. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
6. [ ] **dev** _(auto)_ — Fixed execution step.
7. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
8. [ ] **open_pr** _(auto)_ — Fixed execution step.
9. [ ] **pr_review** _(auto)_ — Fixed execution step.
10. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **architect** — No new components, contracts, or cross-repo changes; just one function in existing file.
- **designer** — Trivial implementation (single function returning dict) needs no file plan.
- **test_architect** — Two simple assertions map directly to the two AC; no edge cases to elaborate.
- **critic** — Single story produces only one artifact to check.

## Planner notes

- attempt 1: Duplicate role: analyst; architect_gate needs ['stories'] but no earlier step produces it (plan must produce stories: analyst `inline_story` or scrum_master)
