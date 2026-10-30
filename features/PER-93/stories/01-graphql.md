# Story — Add version endpoint helper to graphql service

**ID:** PER-93  
**Type:** Story  
**Repo:** adlc-graphql  
**Status:** Draft  
**Complexity:** Trivial  
**Track:** Quick

---

## Overview

Add a `version()` function to the adlc-graphql service that returns the service name and version read from a `VERSION` file at the repository root. This is a backend utility function with no HTTP routes or GraphQL schema changes.

---

## Acceptance Criteria

### AC1: version() returns correct version when VERSION file exists

**Given** a `VERSION` file at the repository root containing `0.1.0`  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.1.0"}`

### AC2: version() returns default when VERSION file is missing

**Given** no `VERSION` file exists at the repository root  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0"}`

### AC3: version() returns default when VERSION file is empty or whitespace-only

**Given** a `VERSION` file at the repository root that is either empty or contains only whitespace  
**When** `version()` is called  
**Then** it returns `{"service": "adlc-graphql", "version": "0.0.0"}`

### AC4: Unit tests cover all acceptance criteria

**Given** test cases in `tests/test_version.py`  
**When** the tests are run  
**Then** AC1, AC2, and AC3 are each verified by at least one test case  
**And** tests use a temporary directory for file manipulation  
**And** the real `VERSION` file at the repository root is never modified during test execution

### AC5: Existing health check tests remain passing

**Given** existing tests in `tests/test_health.py`  
**When** the test suite is run after this feature is implemented  
**Then** all tests in `tests/test_health.py` pass  
**And** `src/health.py` remains unchanged

---

## Technical Specification

### Files to Create

#### `/VERSION`

Plain text file at repository root:

```
0.1.0
```

#### `src/version.py`

New module containing the `version()` function:

```python
"""Version reporting for adlc-graphql service."""

from pathlib import Path
from typing import Dict


def version() -> Dict[str, str]:
    """
    Return the service name and version.
    
    Reads version from VERSION file at repository root.
    Returns {"service": "adlc-graphql", "version": "0.0.0"} if:
    - VERSION file does not exist
    - VERSION file is empty
    - VERSION file contains only whitespace
    
    Returns:
        Dictionary with "service" and "version" keys
    """
    version_file = Path(__file__).parent.parent / "VERSION"
    
    try:
        content = version_file.read_text().strip()
        version_value = content if content else "0.0.0"
    except FileNotFoundError:
        version_value = "0.0.0"
    
    return {
        "service": "adlc-graphql",
        "version": version_value
    }
```

**Implementation notes:**

- Uses `pathlib.Path` to resolve repository root relative to module location
- `read_text()` reads file content as string
- `.strip()` removes leading/trailing whitespace
- Empty string after strip is treated as missing version
- `FileNotFoundError` is caught and results in default version
- Always returns dictionary with both "service" and "version" keys

#### `tests/test_version.py`

Unit tests covering all acceptance criteria:

```python
"""Tests for version module."""

import tempfile
from pathlib import Path
from unittest.mock import patch

import pytest

from src.version import version


def test_version_with_valid_file():
    """AC1: Returns correct version when VERSION file exists with content."""
    with tempfile.TemporaryDirectory() as tmpdir:
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("0.1.0")
        
        with patch("src.version.Path") as mock_path:
            mock_path.return_value.parent.parent = Path(tmpdir)
            result = version()
        
        assert result == {"service": "adlc-graphql", "version": "0.1.0"}


def test_version_with_missing_file():
    """AC2: Returns default version when VERSION file is missing."""
    with tempfile.TemporaryDirectory() as tmpdir:
        # No VERSION file created
        
        with patch("src.version.Path") as mock_path:
            mock_path.return_value.parent.parent = Path(tmpdir)
            result = version()
        
        assert result == {"service": "adlc-graphql", "version": "0.0.0"}


def test_version_with_empty_file():
    """AC3: Returns default version when VERSION file is empty."""
    with tempfile.TemporaryDirectory() as tmpdir:
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("")
        
        with patch("src.version.Path") as mock_path:
            mock_path.return_value.parent.parent = Path(tmpdir)
            result = version()
        
        assert result == {"service": "adlc-graphql", "version": "0.0.0"}


def test_version_with_whitespace_only_file():
    """AC3: Returns default version when VERSION file contains only whitespace."""
    with tempfile.TemporaryDirectory() as tmpdir:
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("   \n\t  ")
        
        with patch("src.version.Path") as mock_path:
            mock_path.return_value.parent.parent = Path(tmpdir)
            result = version()
        
        assert result == {"service": "adlc-graphql", "version": "0.0.0"}


def test_version_with_whitespace_around_content():
    """Returns trimmed version when VERSION file has leading/trailing whitespace."""
    with tempfile.TemporaryDirectory() as tmpdir:
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("  0.1.0\n")
        
        with patch("src.version.Path") as mock_path:
            mock_path.return_value.parent.parent = Path(tmpdir)
            result = version()
        
        assert result == {"service": "adlc-graphql", "version": "0.1.0"}
```

**Test design notes:**

- Each test uses `tempfile.TemporaryDirectory()` to create isolated test environment (AC4)
- `unittest.mock.patch` redirects `Path` resolution to temporary directory
- Real `VERSION` file at repository root is never touched (AC4)
- Five test cases:
  - `test_version_with_valid_file` → AC1
  - `test_version_with_missing_file` → AC2
  - `test_version_with_empty_file` → AC3 (empty case)
  - `test_version_with_whitespace_only_file` → AC3 (whitespace case)
  - `test_version_with_whitespace_around_content` → edge case (valid content with surrounding whitespace)

### Files to Leave Unchanged

- `src/health.py` — existing health check module (AC5)
- `tests/test_health.py` — existing health check tests (AC5)

### Verification Steps

1. Create `/VERSION` file with content `0.1.0`
2. Create `src/version.py` module with `version()` function
3. Create `tests/test_version.py` with five test cases
4. Run `pytest tests/test_version.py` — all five tests pass (AC4)
5. Run `pytest tests/test_health.py` — all existing tests pass (AC5)
6. Import and call `version()` in Python REPL:
   ```python
   from src.version import version
   version()
   # Expected: {'service': 'adlc-graphql', 'version': '0.1.0'}
   ```

---

## Dependencies

None.

---

## Out of Scope

- HTTP routes or GraphQL resolvers that expose the version
- GraphQL schema changes
- Dynamic version resolution from git tags or `pyproject.toml`
- Version validation or format constraints beyond string reading

---

## Notes

This is a utility function only. Future work may add a GraphQL query or REST endpoint that calls `version()`, but that is not part of this story.
