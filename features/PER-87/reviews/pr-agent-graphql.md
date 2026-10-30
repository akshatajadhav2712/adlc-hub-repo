```json
[
  {
    "path": "tests/test_health.py",
    "line": 14,
    "body": "**Duplicate test coverage:** `test_health_returns_ok_status()` (lines 14-17) duplicates `test_health_default()` (lines 8-11). Both verify `health()` returns `{\"status\": \"ok\"}`. Per AC6, the existing test should be preserved and new tests added — one of these is redundant. Remove `test_health_returns_ok_status()` and keep `test_health_default()` as the backward compatibility verification."
  }
]
```
