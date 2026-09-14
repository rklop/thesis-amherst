# test_parsing_rfc850_year_in_past

- **Instance:** `django__django-11848`
- **Test ID:** `django__django-11848--b13200b8ee25f2cb`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

A two-digit RFC 850 year already in the current century's past must remain there. Only a year appearing more than 50 years in the future is rolled back; old past years aren't symmetrically rolled forward.

## Expected behavior

parse_http_date() returns the timestamp for 2020-11-06 08:49:37 UTC, not 2120-11-06 08:49:37 UTC.

## Test command

`cd /testbed && python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_in_past`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
