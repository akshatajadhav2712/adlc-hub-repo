---
key: PER-87
type: story
status: todo
blocks: []
blocked_by: []
labels:
- repo:graphql
repos:
- graphql
summary: Add input validation for health check status values
---

The health check currently always returns {"status": "ok"}. We want it to report

a real status and reject invalid values, so monitoring can trust it.

Add a module `src/status.py` with:

* An allowed set of statuses: "ok", "degraded", "down".
* validate_status(value: str) -> str: returns the value lower-cased and stripped

  if it is allowed; otherwise raises ValueError with the message

  "invalid status: <value>".

Update src/health.py:

* health(status: str = "ok") -> dict returns {"status": validate_status(status)}.
* Calling health() with no argument must still return {"status": "ok"} (backward compatible).

Acceptance criteria:

* AC1: validate_status("ok"), ("degraded"), ("down") return the same value.
* AC2: validate_status("  OK ") returns "ok".
* AC3: validate_status("broken") raises ValueError("invalid status: broken").
* AC4: health() returns {"status": "ok"}; health("down") returns {"status": "down"}.
* AC5: health("broken") raises ValueError.
* AC6: Tests in tests/test_status.py and tests/test_health.py cover AC1–AC5; existing health test still passes.

Notes for spec review: the error message format is part of the contract;

document it in the spec.
