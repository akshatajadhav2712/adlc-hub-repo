# Story — Add input validation for health check status values

**Story ID:** PER-87  
**Type:** Story  
**Epic:** (none)  
**Complexity:** Simple  
**Status:** Draft

---

## Description

Add input validation to the health check endpoint so it can report real status values ("ok", "degraded", "down", "failed") and reject invalid ones. This ensures monitoring systems can trust the status reported by the service.

Currently, the health check always returns `{"status": "ok"}`. This story adds a validation layer that:
- Defines an explicit set of allowed status values (four values: "ok", "degraded", "down", "failed")
- Normalizes input (case-insensitive, whitespace-tolerant)
- Rejects invalid values with a contract-defined error message
- Maintains backward compatibility for existing callers

---

## Acceptance Criteria

### AC1: Valid statuses are accepted unchanged
**Given** the status validator is called with "ok", "degraded", "down", or "failed"  
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
**And when** called with status="failed"  
**Then** it returns `{"status": "failed"}`

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

## Technical Specification

### 1. Status Validation Module

**File:** `src/status.py`

Create a new module containing:

```python
# Allowed status values
ALLOWED_STATUSES = {"ok", "degraded", "down", "failed"}

def validate_status(value: str) -> str:
    """
    Validate and normalize a status value.
    
    Args:
        value: Status string to validate
        
    Returns:
        Normalized status (lowercased and stripped)
        
    Raises:
        ValueError: If status is not in ALLOWED_STATUSES
    """
    normalized = value.strip().lower()
    if normalized not in ALLOWED_STATUSES:
        raise ValueError(f"invalid status: {value}")
    return normalized
```

**Implementation notes:**
- The allowed set is a module-level constant containing four values: "ok", "degraded", "down", "failed"
- Normalization (`.strip().lower()`) happens before validation
- The error message format `"invalid status: <value>"` is part of the contract and must use the original input value, not the normalized one
- The function signature is explicit: takes `str`, returns `str`, raises `ValueError`

---

### 2. Health Module Update

**File:** `src/health.py`

Update the existing `health()` function:

```python
from src.status import validate_status

def health(status: str = "ok") -> dict:
    """
    Return health check response with validated status.
    
    Args:
        status: Health status (default: "ok")
        
    Returns:
        Dictionary with validated status
        
    Raises:
        ValueError: If status is invalid
    """
    return {"status": validate_status(status)}
```

**Implementation notes:**
- Import `validate_status` from the new `src.status` module
- Default argument `status="ok"` preserves backward compatibility
- The function delegates validation entirely to `validate_status()`
- Validation errors propagate naturally (no try/catch needed)

---

### 3. Test Coverage

#### File: `tests/test_status.py` (new)

Create comprehensive tests for the validation module:

```python
import pytest
from src.status import validate_status

def test_valid_statuses_unchanged():
    """AC1: Valid statuses are accepted unchanged"""
    assert validate_status("ok") == "ok"
    assert validate_status("degraded") == "degraded"
    assert validate_status("down") == "down"
    assert validate_status("failed") == "failed"

def test_case_insensitive_with_whitespace():
    """AC2: Case-insensitive and whitespace-tolerant"""
    assert validate_status("  OK ") == "ok"
    assert validate_status("DEGRADED") == "degraded"
    assert validate_status("\tDown\n") == "down"
    assert validate_status("  FAILED  ") == "failed"

def test_invalid_status_raises_value_error():
    """AC3: Invalid statuses raise ValueError with specific message"""
    with pytest.raises(ValueError, match="invalid status: broken"):
        validate_status("broken")
    
    with pytest.raises(ValueError, match="invalid status: unknown"):
        validate_status("unknown")
```

**Test design notes:**
- Each test maps directly to one acceptance criterion
- AC1 test covers all four allowed values
- AC2 test includes variations for all four values: leading/trailing spaces, uppercase, mixed whitespace
- AC3 test verifies both the exception type and the exact message format
- Use `pytest.raises` with `match` parameter to validate error message format

---

#### File: `tests/test_health.py` (update)

Add tests for the updated health function:

```python
import pytest
from src.health import health

# Existing test (must continue to pass)
def test_health_default():
    """Existing test: health() returns ok"""
    result = health()
    assert result == {"status": "ok"}

# New tests for AC4 and AC5
def test_health_with_explicit_status():
    """AC4: Health function uses validated status"""
    assert health("down") == {"status": "down"}
    assert health("degraded") == {"status": "degraded"}
    assert health("failed") == {"status": "failed"}
    assert health("  OK ") == {"status": "ok"}  # Normalization works through health()

def test_health_with_invalid_status():
    """AC5: Health function propagates validation errors"""
    with pytest.raises(ValueError, match="invalid status: broken"):
        health("broken")
```

**Test design notes:**
- Preserve the existing test to verify backward compatibility (AC6)
- AC4 test covers both the default case and explicit status values, including the new "failed" status
- AC5 test verifies that validation errors propagate from `validate_status()` through `health()`
- The AC4 test includes a normalization case to confirm the integration works end-to-end

---

## Implementation Order

1. Create `src/status.py` with `ALLOWED_STATUSES` (four values) and `validate_status()`
2. Create `tests/test_status.py` with AC1–AC3 test cases (including "failed" coverage)
3. Run status tests to verify validation logic
4. Update `src/health.py` to import and use `validate_status()`
5. Update `tests/test_health.py` with AC4–AC5 test cases (including "failed" coverage)
6. Run full test suite to verify AC6 (all tests pass including existing test)

---

## Verification

**Test execution:**
```bash
pytest tests/test_status.py -v  # Verify AC1–AC3
pytest tests/test_health.py -v  # Verify AC4–AC5 and backward compatibility
pytest tests/ -v                # Verify AC6 (complete coverage)
```

**Expected outcomes:**
- `test_valid_statuses_unchanged` passes (all four values) → AC1 verified
- `test_case_insensitive_with_whitespace` passes (all four values) → AC2 verified
- `test_invalid_status_raises_value_error` passes → AC3 verified
- `test_health_with_explicit_status` passes (including "failed") → AC4 verified
- `test_health_with_invalid_status` passes → AC5 verified
- `test_health_default` (existing) passes → Backward compatibility confirmed
- All tests pass → AC6 verified

---

## Technical Notes

### Contract Elements

**Error message format:**
The error message `"invalid status: <value>"` is part of the contract. This format must be preserved because:
- Monitoring systems may parse this message
- The message uses the original input value (not normalized) for debugging clarity
- Future changes to validation logic must maintain this format

**Allowed statuses:**
The set `{"ok", "degraded", "down", "failed"}` is fixed for this story. These values were chosen to represent:
- `ok`: System operating normally
- `degraded`: System functional but with reduced capacity or performance
- `down`: System unavailable or non-functional
- `failed`: Validation failure or critical error state

The addition of "failed" was incorporated per architect review feedback to provide a distinct status for validation failures.

### Backward Compatibility

The default argument `status="ok"` in `health()` ensures that existing callers using `health()` with no arguments continue to work unchanged. The existing test `test_health_default()` verifies this compatibility.

### File Paths (Repository Context)

All file paths are relative to repository root `/home/aksj/Documents/adlc-graphql/`:

**New files:**
- `src/status.py` (validation module)
- `tests/test_status.py` (validation tests)

**Modified files:**
- `src/health.py` (add validation)
- `tests/test_health.py` (add AC4–AC5 tests)

**Existing files preserved:**
- `src/__init__.py`
- `tests/__init__.py`
- `README.md`
- All other files in repo tree remain unchanged

---

## Dependencies

None. This is a self-contained addition to the existing codebase with no external dependencies.

---

## Out of Scope

- Framework or HTTP layer integration (not specified in ticket)
- Logging or monitoring instrumentation
- Status persistence or state management
- Additional status values beyond the four specified
- Status transition logic or state machine behavior
- API documentation updates (not mentioned in requirements)

---

## Notes

- This story implements the full validation contract specified in the ticket
- The error message format is documented per the ticket's spec review note
- Test coverage maps 1:1 to acceptance criteria as required by AC6
- Backward compatibility is explicitly tested and preserved
- The "failed" status was added per architect review feedback and is now covered in all validation logic, acceptance criteria, and test requirements
