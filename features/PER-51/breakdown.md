# Breakdown — PER-51

**Epic:** Choose the caller-facing readiness word-v3

**Product decision required:** Product must choose between "ready" and "available" before implementation begins.

**Implementation scope after decision:** Add exactly one function to `src/health.py` in the `graphql` repo.

---

## Epic acceptance criteria

1. **AC1 (REQ-1):** Product has chosen exactly one word ("ready" or "available") for caller-facing readiness status.
2. **AC2 (REQ-2):** The new function in `src/health.py` returns `{"<chosen_word>": "success"}`.
3. **AC3 (REQ-3):** The existing `health()` function still returns exactly `{"status": "ok"}`.

---

## Execution order

This epic has **2 stories** executed in sequence:

1. **Story 1:** Product decision — choose readiness word (prerequisite for Story 2)
2. **Story 2:** Implement chosen readiness function in health.py

---

## Story 1: Product decision — choose readiness word

**Repo:** `graphql`

**Description:**  
Product reviews the two candidate words ("ready" and "available") and documents their choice. This is a decision-only story; no code is written. The chosen word will be used in Story 2.

**Acceptance criteria:**
- **AC1 (REQ-1):** Product has chosen exactly one word ("ready" or "available") for caller-facing readiness status.

**Dependencies:**  
None (first story in epic).

**Allowed paths:**  
No code changes. Decision documented in ticket or requirements artifact.

**Verification approach:**  
- **Test count:** 0 (decision-only; no automated tests)
- **Verification method:** Manual confirmation that product has documented their choice in the ticket or requirements.

**Prerequisites:**  
None.

**Notes:**  
- This is a human-in-the-loop (HITL) story requiring product sign-off.
- Implementation (Story 2) is blocked until this decision is made.

---

## Story 2: Implement chosen readiness function in health.py

**Repo:** `graphql`

**Description:**  
Add exactly one new function to `src/health.py` that returns the product-chosen readiness word in the success-key format `{"<chosen_word>": "success"}`. The existing `health()` function must remain unchanged and continue to return `{"status": "ok"}`.

**Acceptance criteria:**
- **AC2 (REQ-2):** The new function in `src/health.py` returns `{"<chosen_word>": "success"}` (where `<chosen_word>` is the word product chose in Story 1).
- **AC3 (REQ-3):** The existing `health()` function still returns exactly `{"status": "ok"}`.

**Dependencies:**  
- **After Story 1:** Product must choose the readiness word before implementation begins.

**Allowed paths:**  
- `src/health.py` — add one new function; do not modify existing `health()` function
- `tests/test_health.py` — add tests for the new function; existing `health()` tests must continue to pass

**Verification approach:**  
- **Test count:** 2 (one test for new function return value, one regression test confirming `health()` unchanged)
- **Verification method:**
  1. Unit test: Call new function and assert return value is `{"<chosen_word>": "success"}` with the actual chosen word.
  2. Regression test: Call `health()` and assert return value is still `{"status": "ok"}`.

**Prerequisites:**  
Story 1 complete (product decision made).

**Notes:**  
- Function name and signature are developer's choice.
- No changes to API endpoints, routes, or other repos.
- No edge case handling beyond basic return value.

---

## Epic verification summary

- **Total story count:** 2
- **Total test count:** 2 (0 in Story 1 + 2 in Story 2)
- **Epic closes when:** Both stories are done — product has chosen the word, the new function exists and returns it in the correct format, and `health()` remains unchanged.

---

## Dependencies

- Story 1 has no dependencies (first in epic).
- Story 2 depends on Story 1 (product decision required before code).

---

## Open questions

None. Product decision is the only unresolved item, and it is explicitly Story 1's deliverable.
