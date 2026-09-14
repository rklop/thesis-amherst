# test_same_connection_and_target_timezone_skips_conversion

- **Instance:** `django__django-11138`
- **Test ID:** `django__django-11138--ed0cbf45824e7d79`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

For MySQL date lookups, conversion must be omitted when the database TIME_ZONE already equals the requested current time zone. This is required because even a no-op CONVERT_TZ() depends on MySQL timezone tables that may be unpopulated.

## Expected behavior

The SQL is DATE(legacy_table.happened_at), with no CONVERT_TZ() call.

## Test command

`python tests/runtests.py backends.mysql.test_operations.DatabaseOperationsTests.test_same_connection_and_target_timezone_skips_conversion`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
