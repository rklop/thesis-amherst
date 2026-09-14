# test_parsing_rfc850_exactly_50_years_ahead

- **Instance:** `django__django-11848`
- **Test ID:** `django__django-11848--be8c43ec158290aa`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The 50-year rule is strict: an RFC 850 date exactly 50 years in the future remains in the current century. A pinned datetime may expose its year as an int subclass, so century calculation should preserve that public result without requiring modulo support.

## Expected behavior

parse_http_date() returns the timestamp for 2099-11-06 08:49:37 UTC. Candidate_b calculates the century using floor division and passes; candidate_a invokes the overridden modulo operation and fails.

## Test command

`./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_exactly_50_years_ahead`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
