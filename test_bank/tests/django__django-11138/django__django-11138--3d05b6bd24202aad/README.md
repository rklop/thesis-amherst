# test_query_datetime_lookup_with_database_timezone

- **Instance:** `django__django-11138`
- **Test ID:** `django__django-11138--3d05b6bd24202aad`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Database TIME_ZONE must affect datetime component lookups such as __hour, not only __date and __time casts. SQLite extraction must interpret stored naive values in the database timezone before converting them to the active timezone.

## Expected behavior

The hour lookup returns exactly one Event. candidate_b converts the stored Bangkok-local value to Nairobi before extracting the hour; candidate_a extracts after treating it as UTC and returns no match.

## Test command

`cd /testbed && python tests/runtests.py timezones.tests.NewDatabaseTests.test_query_datetime_lookup_with_database_timezone --parallel 1 --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
