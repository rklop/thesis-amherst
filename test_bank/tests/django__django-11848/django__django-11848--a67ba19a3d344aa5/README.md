# test_parsing_rfc850_year_at_century_boundary

- **Instance:** `django__django-11848`
- **Test ID:** `django__django-11848--a67ba19a3d344aa5`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Two-digit RFC 850 years must be anchored to the current century, not hard-coded to the 2000s. At the century boundary, a year exactly 50 years ahead remains in the future because only dates more than 50 years ahead roll back.

## Expected behavior

The returned timestamp represents 2150-11-06 08:49:37 UTC. candidate_a instead produces 2050 because it always adds 2000.

## Test command

`cd /testbed && python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_at_century_boundary`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
