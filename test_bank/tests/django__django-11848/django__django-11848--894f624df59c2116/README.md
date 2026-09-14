# test_parsing_rfc850_fifty_year_century_boundary

- **Instance:** `django__django-11848`
- **Test ID:** `django__django-11848--894f624df59c2116`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Rollback applies only when an RFC 850 year is more than 50 years in the future. The generated candidate incorrectly maps `20` to 1920 when the current year is 1970, although 2020 is exactly 50 years ahead and must remain unchanged.

## Expected behavior

`parse_http_date()` returns an epoch value representing 2020-11-06 08:49:37 UTC, not 1920.

## Test command

`cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_fifty_year_century_boundary`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
