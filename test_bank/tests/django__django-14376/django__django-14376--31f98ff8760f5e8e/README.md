# backends.mysql.tests.DatabaseWrapperTests.test_empty_options_preserve_top_level_credentials

- **Instance:** `django__django-14376`
- **Test ID:** `django__django-14376--31f98ff8760f5e8e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When OPTIONS contains no database or password override, the MySQL backend must preserve NAME and PASSWORD as the values for the non-deprecated database and password connection parameters. Absence of an override is not equivalent to an explicit None override.

## Expected behavior

get_connection_params() returns database='configured_db' and password='secret'. candidate_a does so; candidate_b overwrites both values with None while processing the empty OPTIONS mapping.

## Test command

`python tests/runtests.py backends.mysql.tests.DatabaseWrapperTests.test_empty_options_preserve_top_level_credentials --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
