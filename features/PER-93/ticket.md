---
key: PER-93
type: story
status: todo
blocks: []
blocked_by: []
labels:
- repo:graphql
repos:
- graphql
summary: Add a version endpoint helper to the graphql service
---

We need a simple way to report which version of the graphql service is running.

Add a function `version()` in a new module `src/version.py` that returns:

{"service": "adlc-graphql", "version": "<value>"}

where <value> is read from a VERSION file at the repo root. If the file is

missing or empty, return "0.0.0".

Add a VERSION file at the repo root containing 0.1.0.

Acceptance criteria:

* AC1: version() returns {"service": "adlc-graphql", "version": "0.1.0"} with the VERSION file present.
* AC2: version() returns version "0.0.0" when VERSION is missing.
* AC3: version() returns version "0.0.0" when VERSION is empty or only whitespace.
* AC4: Unit tests in tests/test_version.py cover AC1–AC3 (use a temp dir, do not edit the real VERSION file).
* AC5: Existing tests (tests/test_health.py) still pass; src/health.py is unchanged.

Out of scope: HTTP routes, GraphQL schema changes.
