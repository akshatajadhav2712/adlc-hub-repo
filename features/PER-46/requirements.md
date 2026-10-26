# Requirements — Add `ready()` beside `health()`

**Feature:** Add ready endpoint function  
**Story ID:** PER-46  
**Epic:** N/A (standalone story)

---

## Overview

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

## Scope

**In scope:**
- Add `ready()` function to `src/health.py`
- Return dictionary `{"status": "ready"}` from `ready()`
- Preserve existing `health()` behavior

**Out of scope:**
- Modifying `health()` function
- Adding API routes or endpoints
- UI changes
- Additional repositories
- Edge case handling beyond the two acceptance criteria

---

## Verification

1. Call `ready()` and assert the return value equals `{"status": "ready"}`
2. Call `health()` and assert the return value equals `{"status": "ok"}`

---

## Notes

- Single-file change in `src/health.py`
- No framework, language, ORM, or cloud stack decisions required
- Product outcome: a ready status check function available for use
