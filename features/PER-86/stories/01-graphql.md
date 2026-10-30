# Story PER-86 — Add version endpoint helper to the graphql service

**Type:** story  
**Epic:** (none)  
**Status:** draft  
**Complexity:** trivial  
**Estimate:** 1 point

---

## Description

Introduce a utility function `version()` in a new module `src/version.py` that reports the service name, version, and stability indicator. The version is read from a `VERSION` file at the repository root (`/home/aksj/Documents/adlc-graphql/VERSION`). If the file is missing or empty, the function returns a fallback version `"0.0.0"`. The function always returns `"stable": "yes"` to indicate production readiness.

A `VERSION` file containing `0.1.0` will be created at the repository root as part of this story.

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
**Then** they verify AC1, AC2, and AC3 using a temporary directory (not editing the real `VERSION` file at `/home/aksj/Documents/adlc-graphql/VERSION`)

---

### AC5: Existing health endpoint tests remain unaffected

**Given** the existing test suite in `tests/test_health.py`  
**When** tests are executed  
**Then** all tests pass and `src/health.py` is unchanged

---

## Technical Specification

### Implementation

**New module:** `src/version.py`

```python
import os
from pathlib import Path

def version() -> dict[str, str]:
    """
    Return the service name, version, and stability indicator.
    
    Version is read from VERSION file at repository root.
    If the file is missing, empty, or whitespace-only, return "0.0.0".
    The "stable" field is always "yes".
    
    Returns:
        dict with keys "service", "version", and "stable"
    """
    # Resolve repository root: parent of src/ directory
    repo_root = Path(__file__).parent.parent
    version_file = repo_root / "VERSION"
    
    version_string = "0.0.0"  # fallback
    
    if version_file.exists():
        content = version_file.read_text().strip()
        if content:
            version_string = content
    
    return {
        "service": "adlc-graphql",
        "version": version_string,
        "stable": "yes"
    }
```

**New file:** `VERSION` (at repository root `/home/aksj/Documents/adlc-graphql/VERSION`)

```
0.1.0
```

**New test module:** `tests/test_version.py`

```python
import tempfile
from pathlib import Path
import sys
import importlib.util

def test_version_with_file_present():
    """AC1: version() returns correct metadata when VERSION file exists."""
    with tempfile.TemporaryDirectory() as tmpdir:
        # Create mock src/version.py in temp directory
        src_dir = Path(tmpdir) / "src"
        src_dir.mkdir()
        
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("0.1.0")
        
        # Load version module with repo_root pointing to tmpdir
        spec = importlib.util.spec_from_file_location(
            "version_test",
            Path(__file__).parent.parent / "src" / "version.py"
        )
        version_module = importlib.util.module_from_spec(spec)
        
        # Patch the module's __file__ to point to temp location
        original_file = version_module.__file__
        version_module.__file__ = str(src_dir / "version.py")
        spec.loader.exec_module(version_module)
        
        result = version_module.version()
        
        assert result == {"service": "adlc-graphql", "version": "0.1.0", "stable": "yes"}


def test_version_with_missing_file():
    """AC2: version() returns fallback when VERSION file is missing."""
    with tempfile.TemporaryDirectory() as tmpdir:
        src_dir = Path(tmpdir) / "src"
        src_dir.mkdir()
        
        # No VERSION file created
        
        spec = importlib.util.spec_from_file_location(
            "version_test",
            Path(__file__).parent.parent / "src" / "version.py"
        )
        version_module = importlib.util.module_from_spec(spec)
        version_module.__file__ = str(src_dir / "version.py")
        spec.loader.exec_module(version_module)
        
        result = version_module.version()
        
        assert result == {"service": "adlc-graphql", "version": "0.0.0", "stable": "yes"}


def test_version_with_empty_file():
    """AC3: version() returns fallback when VERSION file is empty or whitespace."""
    with tempfile.TemporaryDirectory() as tmpdir:
        src_dir = Path(tmpdir) / "src"
        src_dir.mkdir()
        
        version_file = Path(tmpdir) / "VERSION"
        version_file.write_text("   \n\t  ")
        
        spec = importlib.util.spec_from_file_location(
            "version_test",
            Path(__file__).parent.parent / "src" / "version.py"
        )
        version_module = importlib.util.module_from_spec(spec)
        version_module.__file__ = str(src_dir / "version.py")
        spec.loader.exec_module(version_module)
        
        result = version_module.version()
        
        assert result == {"service": "adlc-graphql", "version": "0.0.0", "stable": "yes"}
```

### File Structure After Implementation

```
/home/aksj/Documents/adlc-graphql/
├── VERSION                          (new, contains "0.1.0")
├── README.md
├── src/
│   ├── __init__.py
│   ├── __pycache__/
│   │   ├── __init__.cpython-312.pyc
│   │   └── health.cpython-312.pyc
│   ├── health.py                    (unchanged)
│   └── version.py                   (new)
└── tests/
    ├── __init__.py
    ├── __pycache__/
    │   ├── __init__.cpython-312.pyc
    │   └── test_health.cpython-312.pyc
    ├── test_health.py               (unchanged)
    └── test_version.py              (new)
```

### Fallback Logic

1. Construct path to `VERSION` file: `<repo_root>/VERSION` where `<repo_root>` is the parent directory of `src/`.
2. Check if file exists using `Path.exists()`.
3. If exists, read content and strip whitespace.
4. If stripped content is non-empty, use it; otherwise use `"0.0.0"`.
5. If file does not exist, use `"0.0.0"`.
6. Always set `"stable"` field to `"yes"`.

### Test Isolation

All tests in `tests/test_version.py` use `tempfile.TemporaryDirectory()` to create isolated filesystem environments. Tests dynamically load the `version.py` module with a patched `__file__` attribute pointing to the temporary directory, ensuring the real `VERSION` file at `/home/aksj/Documents/adlc-graphql/VERSION` is never read or modified during test execution.

---

## Out of Scope

- Adding HTTP routes or GraphQL schema changes to expose the version endpoint
- Modifying any existing modules beyond creating the new files listed above
- Dynamic or configurable stability indicators (the `"stable"` field is hardcoded to `"yes"`)

---

## Dependencies

None

---

## Tests

### Unit Tests (AC4)

All acceptance criteria AC1–AC3 are verified by unit tests in `tests/test_version.py`:

- `test_version_with_file_present()` — verifies AC1
- `test_version_with_missing_file()` — verifies AC2  
- `test_version_with_empty_file()` — verifies AC3

Tests use temporary directories to ensure isolation from the production `VERSION` file.

### Regression Tests (AC5)

Existing tests in `tests/test_health.py` must continue to pass without modification. `src/health.py` remains unchanged.

---

## Notes

- The `VERSION` file path is `/home/aksj/Documents/adlc-graphql/VERSION` (repository root).
- The `"stable"` field is always set to `"yes"` to indicate production readiness.
- This story implements only the utility function; integration into HTTP or GraphQL endpoints is deferred to future work.
- Test isolation is critical: tests must not depend on or modify the real `VERSION` file.
