# Requirements — PER-50

**Ticket:** Choose the caller-facing readiness word-v2  
**Epic:** *(none — standalone story)*  
**Status:** draft  
**Complexity:** trivial

---

## Problem Statement

The GraphQL service needs a caller-facing readiness indicator. Product has chosen **"available"** as the readiness word. Implement exactly one new function in `src/health.py` that returns this word. The existing `health()` function must remain unchanged.

---

## Scope

**In scope:**
- One new function in `src/health.py` returning `{"status": "available"}`

**Out of scope:**
- Changes to `health()` function
- New API endpoints
- UI components
- Multiple functions or additional design documents
- Framework, language, ORM, or cloud stack decisions

---

## Acceptance Criteria

### AC1: New Function Returns Available

**Given** the new function is called  
**When** it executes  
**Then** it returns `{"status": "available"}`

### AC2: Existing Function Unchanged

**Given** the existing `health()` function in `src/health.py`  
**When** the new function is added  
**Then** `health()` still returns exactly `{"status": "ok"}` with no modifications

---

## Dependencies

- **Repository:** `graphql` (`/home/aksj/Documents/adlc-graphql`)
- **Modified file:** `src/health.py` (one new function only)
- **Test file:** `tests/test_health.py` (two assertions: new function returns `{"status": "available"}`, `health()` unchanged)

---

## Success Criteria

- One new function exists in `src/health.py` returning `{"status": "available"}`
- `health()` function remains exactly as-is
- Tests verify both the new function output and `health()` immutability

---

## Open Questions

*(none)*

---

## Notes

- This is a single-story, single-repo change with no architectural decisions
- Product has chosen "available" as the readiness word via review feedback
- No frameworks, ORMs, or technical stack choices are needed
