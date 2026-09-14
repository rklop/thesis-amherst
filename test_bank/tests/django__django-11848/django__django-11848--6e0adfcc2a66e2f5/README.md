# test_parsing_rfc850_year_exactly_50_years_in_future

- **Instance:** `django__django-11848`
- **Test ID:** `django__django-11848--6e0adfcc2a66e2f5`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

A two-digit RFC 850 year exactly 50 years in the future must remain future-facing even when it crosses a century boundary; only dates more than 50 years ahead roll into the past.

## Expected behavior

`parse_http_date()` returns a timestamp corresponding to 2019-01-01 00:00:00 UTC. Candidate_a instead resolves the year to 1919 by anchoring it to the current century.

## Test command

`cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_exactly_50_years_in_future`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
