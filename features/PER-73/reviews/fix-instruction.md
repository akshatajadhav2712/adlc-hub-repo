Fix these forge PR review comments (auto-route):

### graphql comment 4400141365
**Bug: Sort key fails when dueAt is invalid ISO-8601.** `parse_iso8601` is called inside `sorted(..., key=lambda item: parse_iso8601(item['dueAt']))`. If a reminder somehow has an invalid `dueAt` (e.g. corrupted in-memory state or concurrency edge case), `parse_iso8601` raises `ValueError` and the sort crashes instead of returning a partial list or skipping the bad entry. AC7 requires stable sorting; this violates it. **Fix:** validate `dueAt` at creation (already done) and guard the sort with a try/except or filter invalid entries before sorting.

### graphql comment 4400141371
**Bug: `fromisoformat` rejects valid ISO-8601 with fractional seconds in Python <3.11.** The code uses `datetime.fromisoformat(normalized)` which prior to Python 3.11 does not parse fractional seconds (e.g. `2025-06-01T12:00:00.123Z`). AC5/AC6 require accepting ISO-8601 dates; if the Python version is <3.11, valid dates are rejected. **Fix:** Use `datetime.strptime` with a format that handles fractional seconds, or document the Python version requirement as >=3.11.

### graphql comment 4400141386
**Test gap: No coverage for the exact "now" edge case.** AC6 says "a past date is rejected," but the test only checks a clearly past date (`2020-01-01`). The validation logic rejects `<=`, so a date exactly equal to "now" is also rejected. No test verifies that a date 1 second in the future is accepted, or that "now" is rejected. **Add test:** call `addReminder` with `datetime.now(timezone.utc).isoformat()` and verify rejection (or acceptance if the spec means "not past" rather than "strictly future"), then verify a date 1 second ahead is accepted.

### graphql comment 4400141393
**Test coverage gap: No test for blank description (whitespace-only).** AC5 says description is optional; Story 2 LLD says "blank values (only spaces) become None." Test `test_add_reminder_with_description` covers a valid description, `test_add_reminder_description_too_long` covers over-limit, but no test covers `description: "   "` to verify it becomes `None` per the LLD. **Add test:** `addReminder(..., description: "   ")` and verify the returned reminder has `description: null`.
