# Requirements — PER-47

## Overview

Product decision required: choose the caller-facing readiness word for a new health check function. Two options: "ready" or "available". Once chosen, implement one function in src/health.py that returns the selected word.

## Acceptance Criteria

### AC1: Product decision recorded

**Given** the two candidate words "ready" and "available"  
**When** product reviews the options  
**Then** product selects exactly one word and records the decision

**Verification:** Decision documented in ticket or requirements with clear rationale for the chosen word.

---

### AC2: New readiness function implemented

**Given** product has chosen the readiness word (either "ready" or "available")  
**When** the new function is called  
**Then** it returns `{"status": "<product-chosen-word>"}`

**Verification:** Function exists in src/health.py and returns the exact dict structure with the product-approved word.

---

### AC3: Existing health function unchanged

**Given** the health() function currently returns `{"status": "ok"}`  
**When** health() is called after the new function is added  
**Then** health() still returns exactly `{"status": "ok"}`

**Verification:** Existing tests for health() pass unchanged; health() behavior is unmodified.

---

## Open Questions

- Which word does product choose: "ready" or "available"?
- What is the new function's name? (e.g., `readiness()`, `caller_status()`)

## Notes

- Scope: One new function in src/health.py (graphql repo only)
- No changes to health()
- No UI, no new API endpoint, no second function
- AC2 cannot be completed until product selects the word
- Implementation is trivial once product decides
