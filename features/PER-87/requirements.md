# Requirements — Add input validation for health check status values

**Story ID:** PER-87  
**Epic:** (none)  
**Type:** Story  
**Complexity:** Simple

---

## Overview

Add input validation to the health check endpoint so it can report real status values ("ok", "degraded", "down") and reject invalid ones. This ensures monitoring systems can trust the status reported by the service.

---

## Acceptance Criteria

### AC1: Valid statuses are accepted unchanged
**Given** the status validator is called with "ok", "degraded", or "down"  
**When** the value is passed to `validate_status()`  
**Then** the same value is returned unchanged

### AC2: Case-insensitive and whitespace-tolerant validation
**Given** the status validator is called with "  OK " (uppercase with surrounding whitespace)  
**When** the value is passed to `validate_status()`  
**Then** "ok" is returned (lowercased and stripped)

### AC3: Invalid statuses raise ValueError with specific message
**Given** the status validator is called with "broken" (not in the allowed set)  
**When** the value is passed to `validate_status()`  
**Then** a `ValueError` is raised with message "invalid status: broken"

### AC4: Health function uses validated status
**Given** the health function is called  
**When** called with no argument  
**Then** it returns `{"status": "ok"}`  
**And when** called with status="down"  
**Then** it returns `{"status": "down"}`

### AC5: Health function propagates validation errors
**Given** the health function is called with an invalid status  
**When** called with status="broken"  
**Then** a `ValueError` is raised (from `validate_status`)

### AC6: Test coverage is complete
**Given** tests are written for the new validation logic  
**When** the test suite runs  
**Then** `tests/test_status.py` covers AC1–AC3  
**And** `tests/test_health.py` covers AC4–AC5  
**And** the existing health test still passes (backward compatibility verified)

---

## Technical Notes

### Status Module (`src/status.py`)
- **Allowed statuses:** `{"ok", "degraded", "down"}`
- **Function:** `validate_status(value: str) -> str`
  - Returns the value lowercased and stripped if it is in the allowed set
  - Raises `ValueError` with message format: `"invalid status: <value>"` if not allowed
  - The error message format is part of the contract and must be preserved

### Health Module Update (`src/health.py`)
- **Function signature:** `health(status: str = "ok") -> dict`
- Returns `{"status": validate_status(status)}`
- Default argument ensures backward compatibility: `health()` returns `{"status": "ok"}`

### Test Files
- `tests/test_status.py` — validation logic (AC1–AC3)
- `tests/test_health.py` — health function behavior (AC4–AC5), including existing test

---

## Implementation Scope

**In scope:**
- Create `src/status.py` with validation logic
- Update `src/health.py` to use `validate_status()`
- Write comprehensive tests covering all acceptance criteria
- Maintain backward compatibility (no-argument call)

**Out of scope:**
- Framework or HTTP layer integration (not specified)
- Logging or monitoring instrumentation
- Additional status values beyond the three specified
- Status persistence or state management

---

## Verification

Each acceptance criterion maps to specific test cases:
- AC1 → `test_status.py`: test valid statuses pass through
- AC2 → `test_status.py`: test case/whitespace normalization
- AC3 → `test_status.py`: test invalid status raises ValueError with correct message
- AC4 → `test_health.py`: test default and explicit status values
- AC5 → `test_health.py`: test invalid status propagates error
- AC6 → Test suite execution confirms all above pass + existing test still passes

---

## Dependencies

None. This is a self-contained addition to the existing codebase.

---

## Notes

- The error message format `"invalid status: <value>"` is documented as part of the contract per the ticket's spec review note
- The allowed set is fixed at three values; future expansion is not in scope for this story
- Backward compatibility is preserved: existing calls to `health()` continue to work
