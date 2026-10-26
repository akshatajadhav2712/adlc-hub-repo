# Plan — PER-50 (v1)

- made from: ticket
- source: LLM planner
- complexity: trivial
- needs plan review: no
- track hint: standard

Product wording decision requiring tpm_gate sign-off, then a single-function implementation with inline story.

## Steps

1. [x] **analyst (requirements)** — Capture the product decision question and post-approval acceptance criteria.
   - focus: Focus on the two wording options (ready vs available) and the single-function requirement; no technical design needed yet.
2. [ ] **tpm_gate (review)** — Product must choose between 'ready' and 'available' before any technical work.
   - focus: Approve one of the two wording options so the story can proceed with concrete acceptance criteria.
3. [ ] **ticketize (normalize)** _(auto)_ — Normalize proposed tickets after TPM approval.
4. [ ] **scrum_master (breakdown)** — Convert the product-approved requirements into an implementable story with the chosen word.
   - focus: Single story for graphql repo; use the product-approved word in AC; single function in src/health.py, no changes to health().
5. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
6. [ ] **dev** _(auto)_ — Fixed execution step.
7. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
8. [ ] **open_pr** _(auto)_ — Fixed execution step.
9. [ ] **pr_review** _(auto)_ — Fixed execution step.
10. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **architect** — Single trivial function in existing file; no contracts, schema, or architectural decisions.
- **designer** — Trivial single-function addition; no file plan or algorithm design required.
- **test_architect** — Two simple assertions (new function returns chosen word, health() unchanged); no edge cases or complex test mapping.
- **critic** — Single requirements document and one story; nothing to cross-check.

## Planner notes

- attempt 1: Duplicate role: analyst; architect_gate needs ['stories'] but no earlier step produces it (plan must produce stories: analyst `inline_story` or scrum_master)
