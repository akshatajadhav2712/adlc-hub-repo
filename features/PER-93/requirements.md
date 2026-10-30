# Requirements — PER-93

**Ticket:** Add a version endpoint helper to the graphql service  
**Repo:** `adlc-graphql`  
**Type:** Feature  
**Complexity:** Trivial

---

## Overview

Add a `version()` function to report the running version of the adlc-graphql service. The function reads from a `VERSION` file at the repository root and returns a JSON-serializable dictionary. This is a backend utility; no HTTP routes or GraphQL schema changes are in scope.

---

## Acceptance Criteria

### AC1: version() returns correct version when VERSION file exists

**Given** a `VERSION` file at the repository root containing `0.1.0`  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.1.0"}`

---

### AC2: version() returns default when VERSION file is missing

**Given** no `VERSION` file exists at the repository root  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0"}`

---

### AC3: version() returns default when VERSION file is empty or whitespace-only

**Given** a `VERSION` file at the repository root that is either empty or contains only whitespace  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0"}`

---

### AC4: Unit tests cover all acceptance criteria

**Given** test cases in `tests/test_version.py`  
**When** the tests are run  
**Then** AC1, AC2, and AC3 are each verified by at least one test case  
**And** tests use a temporary directory for file manipulation  
**And** the real `VERSION` file at the repository root is never modified during test execution

---

### AC5: Existing health check tests remain passing

**Given** existing tests in `tests/test_health.py`  
**When** the test suite is run after this feature is implemented  
**Then** all tests in `tests/test_health.py` pass  
**And** `src/health.py` remains unchanged

---

## Implementation Notes

- **New module:** `src/version.py` — contains the `version()` function
- **New file:** `VERSION` at repository root — contains `0.1.0`
- **New test file:** `tests/test_version.py` — unit tests for `version()`
- **Unchanged:** `src/health.py`, `tests/test_health.py`

The `version()` function:

1. Reads from `<repo_root>/VERSION`
2. Strips whitespace from the file content
3. Returns `"0.0.0"` when the file is missing, empty, or whitespace-only
4. Returns the file content as the version value otherwise
5. Always returns `{"service": "adlc-graphql", "version": "<value>"}`

---

## Out of Scope

- HTTP routes or GraphQL schema changes
- Exposing the version via any endpoint
- Dynamic version resolution from git tags or package metadata
