# Story — Add `ready()` beside `health()`

**ID:** PER-46  
**Type:** Story  
**Epic:** N/A (standalone story)  
**Status:** Draft  
**Track:** Quick  
**Complexity:** Trivial

---

## Description

Add a new `ready()` function to `src/health.py` that returns `{"status": "ready"}`. The existing `health()` function remains unchanged and continues to return `{"status": "ok"}`.

---

## Acceptance Criteria

### AC1: ready() returns the correct status
**Given** the `ready()` function is called  
**When** it executes  
**Then** it returns exactly `{"status": "ready"}`

### AC2: health() remains unchanged
**Given** the `health()` function is called  
**When** it executes  
**Then** it still returns exactly `{"status": "ok"}`

---

## Technical Specification

### Implementation

**File:** `src/health.py`

Add the following function to the existing file without modifying the `health()` function:

```python
def ready():
    return {"status": "ready"}
```

The `health()` function remains:

```python
def health():
    return {"status": "ok"}
```

---

## Tests

**File:** `tests/test_health.py`

Add two test cases to verify the acceptance criteria:

```python
def test_ready_returns_ready_status():
    """AC1: ready() returns exactly {"status": "ready"}"""
    from src.health import ready
    assert ready() == {"status": "ready"}

def test_health_still_returns_ok_status():
    """AC2: health() still returns exactly {"status": "ok"}"""
    from src.health import health
    assert health() == {"status": "ok"}
```

---

## Verification

| AC | Verification Method |
|----|---------------------|
| AC1 | `test_ready_returns_ready_status()` passes |
| AC2 | `test_health_still_returns_ok_status()` passes |

Run: `pytest tests/test_health.py -v`

---

## Notes

- Single-file change in `src/health.py`
- No API routes, UI, or framework changes
- Two functions, two tests, one file
