```json
[
  {
    "path": "src/store.py",
    "line": 84,
    "body": "**Bug: Sort key fails when dueAt is invalid ISO-8601.** `parse_iso8601` is called inside `sorted(..., key=lambda item: parse_iso8601(item['dueAt']))`. If a reminder somehow has an invalid `dueAt` (e.g. corrupted in-memory state or concurrency edge case), `parse_iso8601` raises `ValueError` and the sort crashes instead of returning a partial list or skipping the bad entry. AC7 requires stable sorting; this violates it. **Fix:** validate `dueAt` at creation (already done) and guard the sort with a try/except or filter invalid entries before sorting."
  },
  {
    "path": "src/validation.py",
    "line": 42,
    "body": "**Bug: `fromisoformat` rejects valid ISO-8601 with fractional seconds in Python <3.11.** The code uses `datetime.fromisoformat(normalized)` which prior to Python 3.11 does not parse fractional seconds (e.g. `2025-06-01T12:00:00.123Z`). AC5/AC6 require accepting ISO-8601 dates; if the Python version is <3.11, valid dates are rejected. **Fix:** Use `datetime.strptime` with a format that handles fractional seconds, or document the Python version requirement as >=3.11."
  },
  {
    "path": "src/validation.py",
    "line": 54,
    "body": "**AC gap: `validate_future_date` uses `<=` instead of `<`.** Product decision 3 says \"dueAt must be in the future,\" which typically means strictly greater than now. Line 55 rejects when `parsed <= datetime.now(timezone.utc)`. A reminder created at the exact same microsecond as \"now\" is rejected even though it's not in the past. Most implementations use `<` for \"not in future\" checks. AC6 edge case \"past date\" expects rejection of `2020-01-01` (clearly past), but the current code also rejects \"right now.\" **Clarify or fix:** change to `parsed < datetime.now(timezone.utc)` if \"future\" means \"strictly after now,\" or document that \"now\" counts as past."
  },
  {
    "path": "tests/test_reminders.py",
    "line": 139,
    "body": "**Test gap: No coverage for the exact \"now\" edge case.** AC6 says \"a past date is rejected,\" but the test only checks a clearly past date (`2020-01-01`). The validation logic rejects `<=`, so a date exactly equal to \"now\" is also rejected. No test verifies that a date 1 second in the future is accepted, or that \"now\" is rejected. **Add test:** call `addReminder` with `datetime.now(timezone.utc).isoformat()` and verify rejection (or acceptance if the spec means \"not past\" rather than \"strictly future\"), then verify a date 1 second ahead is accepted."
  },
  {
    "path": "tests/test_reminders.py",
    "line": 339,
    "body": "**Test coverage gap: No test for blank description (whitespace-only).** AC5 says description is optional; Story 2 LLD says \"blank values (only spaces) become None.\" Test `test_add_reminder_with_description` covers a valid description, `test_add_reminder_description_too_long` covers over-limit, but no test covers `description: \"   \"` to verify it becomes `None` per the LLD. **Add test:** `addReminder(..., description: \"   \")` and verify the returned reminder has `description: null`."
  },
  {
    "path": "src/schema.py",
    "line": 1,
    "body": "**Process note: No executable entry point or server setup.** The schema and resolvers are complete, but the repo has no `main.py`, `app.py`, or server invocation. The HLD and stories do not specify how the GraphQL endpoint is exposed (HTTP server, CLI, etc.). This is not a bug in the code, but the feature cannot be used outside of tests. **Recommendation:** add a minimal server (e.g. Flask + graphql-server-core) or document that the feature is test-only for now."
  }
]
```
