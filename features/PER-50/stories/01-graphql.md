# Story — Choose the caller-facing readiness word-v2

**ID:** PER-50  
**Type:** story  
**Epic:** *(standalone)*  
**Repository:** graphql (`/home/aksj/Documents/adlc-graphql`)  
**Complexity:** trivial  
**Status:** draft

---

## Overview

Implement a new function in `src/health.py` that returns the product-approved readiness word **"available"** for callers of the GraphQL service. The existing `health()` function remains unchanged.

---

## Requirements Reference

- **Source:** Requirements — PER-50
- **Product decision:** "available" chosen as the readiness word (via review feedback)

---

## Technical Specification

### Modified Files

**`src/health.py`**

Add one new function:

```python
def readiness():
    """Return caller-facing readiness status."""
    return {"status": "available"}
```

The existing `health()` function **must not be modified**. It continues to return `{"status": "ok"}`.

---

### Tests

**`tests/test_health.py`**

Add two test assertions:

1. **New function returns "available":**
   ```python
   def test_readiness_returns_available():
       from src.health import readiness
       assert readiness() == {"status": "available"}
   ```

2. **Existing function unchanged:**
   ```python
   def test_health_unchanged():
       from src.health import health
       assert health() == {"status": "ok"}
   ```

---

## Acceptance Criteria

### AC1: New Function Returns Available

**Given** the new `readiness()` function is called  
**When** it executes  
**Then** it returns `{"status": "available"}`

**Verification:**
- Run `tests/test_health.py::test_readiness_returns_available`
- Assert output is `{"status": "available"}`

---

### AC2: Existing Function Unchanged

**Given** the existing `health()` function in `src/health.py`  
**When** the new `readiness()` function is added  
**Then** `health()` still returns exactly `{"status": "ok"}` with no modifications

**Verification:**
- Run `tests/test_health.py::test_health_unchanged`
- Assert `health()` output is `{"status": "ok"}`
- Inspect `src/health.py` to confirm `health()` code is unmodified

---

## Implementation Notes

- **One function only:** `readiness()` in `src/health.py`
- **No changes to `health()`** — it remains exactly as-is
- **No new API endpoints** — this is an internal function addition only
- **No UI, no framework changes** — trivial single-function implementation

---

## Dependencies

- Repository: `graphql` (`/home/aksj/Documents/adlc-graphql`)
- Existing file: `src/health.py`
- Test file: `tests/test_health.py`

---

## Definition of Done

- [ ] `readiness()` function added to `src/health.py` returning `{"status": "available"}`
- [ ] `health()` function unmodified
- [ ] Both test assertions pass in `tests/test_health.py`
- [ ] Code review approved
- [ ] PR merged
