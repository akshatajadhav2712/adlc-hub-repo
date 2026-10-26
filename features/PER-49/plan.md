# Plan — PER-49

- source: LLM planner
- complexity: simple
- track hint: standard

Single function addition in one repo requiring product sign-off on a word choice before implementation.

## Steps

1. [x] **analyst (requirements)** — Capture the two-option choice and acceptance criteria as Given/When/Then requirements before product review.
   - focus: Document the two options (ready vs available) clearly and write acceptance criteria that use a placeholder for the chosen word.
2. [ ] **tpm_gate (review)** — Product must choose between ready and available before any technical work.
3. [ ] **ticketize (normalize)** _(auto)_ — Normalize proposed tickets after TPM approval.
4. [ ] **scrum_master (breakdown)** — After product choice, create one implementable story with the chosen word.
   - focus: Use the product-approved word from tpm_gate; single story for src/health.py function addition.
5. [ ] **architect_gate (review)** _(auto)_ — Human approval of specs and this plan before Dev.
6. [ ] **dev** _(auto)_ — Fixed execution step.
7. [ ] **pre_pr_qa** _(auto)_ — Fixed execution step.
8. [ ] **open_pr** _(auto)_ — Fixed execution step.
9. [ ] **pr_review** _(auto)_ — Fixed execution step.
10. [ ] **merge** _(auto)_ — Fixed execution step.

## Skipped roles

- **architect** — No new components, API contracts, or cross-repo changes; trivial function addition.
- **designer** — Function implementation is straightforward; no file plan or algorithm design required.
- **test_architect** — Two acceptance criteria with no edge cases; scrum_master story AC is sufficient.
- **critic** — Only requirements and one story; minimal artifacts to cross-check.

## Planner notes

- attempt 1: Duplicate role: analyst; architect_gate needs ['stories'] but no earlier step produces it (plan must produce stories: analyst `inline_story` or scrum_master)
