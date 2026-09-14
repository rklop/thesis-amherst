# test_query_time_lookup

- **Instance:** `django__django-11138`
- **Test ID:** `django__django-11138--2edeb77d6223ea3a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Adding database-time-zone support to SQLite’s date lookup must preserve the adjacent public DateTimeField `__time` lookup. The generated candidate changes that lookup’s registered SQL-function arity without changing its caller.

## Expected behavior

The queryset’s `exists()` result is True. Evaluation completes normally rather than raising a SQLite wrong-number-of-arguments error.

## Test command

`cd /testbed && python tests/runtests.py timezones.tests.NewDatabaseTests.test_query_time_lookup --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
