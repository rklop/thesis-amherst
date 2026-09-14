# test_iso_year_lookup_bounds_for_date_field

- **Instance:** `django__django-14170`
- **Test ID:** `django__django-14170--607f790bf88d22c9`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

ISO-year lookup bounds must cross calendar-year boundaries when necessary and, like ordinary year lookup bounds, must be returned in the backend-adapted representation rather than as raw Python date objects.

## Expected behavior

The method returns the two backend-adapted date values ['2014-12-29', '2016-01-03']. candidate_b applies adapt_datefield_value() to both bounds; candidate_a returns datetime.date objects instead.

## Test command

`python tests/runtests.py backends.base.test_operations.SimpleDatabaseOperationTests.test_iso_year_lookup_bounds_for_date_field`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
