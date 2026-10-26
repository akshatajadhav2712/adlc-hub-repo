# Requirements — PER-49

## Context

The graphql service needs a new health check function in `src/health.py` to check service health status. Product must choose between two status word options: "ready" or "available". No implementation work begins until product sign-off on the word choice.

## Scope

- **In scope**: One new function in `src/health.py` returning a status dictionary to check service health status
- **Out of scope**: Changes to existing `health()` function, new APIs, UI changes, additional repositories

## Acceptance Criteria

### AC1: Product chooses status word

**Given** the graphql service needs a new health check function to check service health status  
**When** product reviews the two options ("ready" and "available")  
**Then** product selects one word and documents the choice in this requirements artifact

**Verification**: Requirements document updated with `[PRODUCT_CHOICE: <word>]` marker

---

### AC2: New function returns chosen status

**Given** product has chosen the status word `<CHOSEN_WORD>`  
**When** the new health check function is called to check service health status  
**Then** it returns `{"status": "<CHOSEN_WORD>"}`

**Verification**: Function execution returns dictionary with "status" key containing the product-approved word

---

### AC3: Existing health function unchanged

**Given** the new health check function exists  
**When** the existing `health()` function is called  
**Then** it still returns `{"status": "ok"}`

**Verification**: Existing `health()` behavior verified through existing test suite; no regression

---

## Open Questions

- **Q1**: What is the intended caller or use case for this new function? (Does not block implementation, but informs function naming)

## Product Decision Required

**Status word options**: "ready" OR "available"

`[PRODUCT_CHOICE: _____]` ← Product to fill before implementation proceeds

---

## Notes

- Single function addition in `src/health.py`
- Purpose: Check service health status
- No changes to existing `health()` function
- No new API endpoints or routes required
- Implementation is one story after product choice
