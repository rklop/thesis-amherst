# test_datetime_cast_date_requires_connection_timezone

- **Instance:** `django__django-11138`
- **Test ID:** `django__django-11138--9ee788e2dbacfa14`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

SQLite timezone-aware datetime SQL functions require both the target timezone and the database connection timezone. Omitting the source database timezone must not silently fall back to UTC.

## Expected behavior

The database raises OperationalError for the invalid two-argument call. candidate_b registers a fixed three-argument function; candidate_a registers a variadic function and incorrectly accepts the call.

## Test command

`./tests/runtests.py backends.sqlite.tests.Tests.test_datetime_cast_date_requires_connection_timezone --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
