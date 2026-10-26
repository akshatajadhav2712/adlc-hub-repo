# Requirements — PER-50

**Ticket:** Choose the caller-facing readiness word-v2  
**Epic:** *(none — standalone story)*  
**Status:** draft  
**Complexity:** trivial

---

## Problem Statement

The GraphQL service needs a caller-facing readiness indicator. Product must choose between two wording options: **"ready"** or **"available"**. No technical implementation can proceed until product sign-off is obtained.

After sign-off, implement exactly one new function in `src/health.py` that returns the chosen word. The existing `health()` function must remain unchanged.

---

## Scope

**In scope:**
- Product decision between "ready" and "available"
- One new function in `src/health.py` returning the chosen status word

**Out of scope:**
- Changes to `health()` function
- New API endpoints
- UI components
- Multiple functions or additional design documents
- Framework, language, ORM, or cloud stack decisions

---

## Acceptance Criteria

### AC1: Product Wording Decision

**Given** the two wording options ("ready" and "available")  
**When** product reviews and approves one option  
**Then** the chosen word is documented and available for implementation

### AC2: New Function Implementation

**Given** product has approved the wording (either "ready" or "available")  
**When** the new function is implemented in `src/health.py`  
**Then** the function returns `{"status": "<product-approved-word>"}`

### AC3: Existing Function Unchanged

**Given** the existing `health()` function in `src/health.py`  
**When** the new function is added  
**Then** `health()` still returns exactly `{"status": "ok"}` with no modifications

---

## Dependencies

- **Blocker:** Product sign-off on wording choice must occur before implementation begins
- **Repository:** `graphql` (`/home/aksj/Documents/adlc-graphql`)
- **Modified file:** `src/health.py` (one new function only)
- **Test file:** `tests/test_health.py` (two assertions: new function returns chosen word, `health()` unchanged)

---

## Success Criteria

- Product has selected and documented either "ready" or "available"
- One new function exists in `src/health.py` returning the approved wording
- `health()` function remains exactly as-is
- Tests verify both the new function output and `health()` immutability

---

## Open Questions

- Which word does product choose: "ready" or "available"?

---

## Notes

- This is a single-story, single-repo change with no architectural decisions
- No frameworks, ORMs, or technical stack choices are needed
- Implementation waits on product approval (tpm_gate)
