# Requirements — Add version endpoint helper

**Ticket:** PER-86  
**Epic:** (none)  
**Product:** adlc-graphql  
**Complexity:** trivial

---

## Summary

Introduce a utility function `version()` in a new module `src/version.py` that reports the service name, version, and stability indicator. The version is read from a `VERSION` file at the repository root. If the file is missing or empty, the function returns a fallback version `"0.0.0"`. The function always returns `"stable": "yes"` to indicate production readiness.

---

## Acceptance Criteria

### AC1: version() returns service metadata when VERSION file is present

**Given** the repository root contains a `VERSION` file with content `0.1.0`  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.1.0", "stable": "yes"}`

---

### AC2: version() returns fallback when VERSION file is missing

**Given** the `VERSION` file does not exist at the repository root  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0", "stable": "yes"}`

---

### AC3: version() returns fallback when VERSION file is empty or whitespace-only

**Given** the `VERSION` file exists but contains only whitespace or is empty  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0", "stable": "yes"}`

---

### AC4: Unit tests cover all version behaviors in isolation

**Given** a test suite in `tests/test_version.py`  
**When** tests are executed  
**Then** they verify AC1, AC2, and AC3 using a temporary directory (not editing the real `VERSION` file)

---

### AC5: Existing health endpoint tests remain unaffected

**Given** the existing test suite in `tests/test_health.py`  
**When** tests are executed  
**Then** all tests pass and `src/health.py` is unchanged

---

## Technical Notes

- **Return value structure:** The `version()` function always returns a dictionary with three keys: `"service"`, `"version"`, and `"stable"`.
- **Stable field:** The `"stable"` field is always set to `"yes"` to indicate the service is in a stable state.
- **File location:** `VERSION` file is located at the repository root (same level as `README.md`).
- **Fallback logic:** If the file is missing, empty, or contains only whitespace, return version `"0.0.0"` while maintaining the `"stable": "yes"` field.
- **Test isolation:** Tests must use a temporary directory to avoid modifying or depending on the actual `VERSION` file.
- **No HTTP or GraphQL changes:** This story implements only the utility function; route and schema integration are out of scope.

---

## Out of Scope

- Adding HTTP routes or GraphQL schema changes to expose the version endpoint.
- Modifying existing modules beyond creating `src/version.py` and `tests/test_version.py`.
- Dynamic or configurable stability indicators (the `"stable"` field is hardcoded to `"yes"`).

---

## Verification

All acceptance criteria (AC1–AC5) are verified by unit tests in `tests/test_version.py` and the continued passing of `tests/test_health.py`.
