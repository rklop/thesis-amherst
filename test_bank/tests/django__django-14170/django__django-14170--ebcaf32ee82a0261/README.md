# test_iso_year_lookup_bounds_for_date_field

- **Instance:** `django__django-14170`
- **Test ID:** `django__django-14170--ebcaf32ee82a0261`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

ISO-year date bounds are returned as native date values, and may cross both adjacent calendar years; backend adaptation is not part of this helper’s output.

## Expected behavior

A list of native datetime.date values: [datetime.date(2019, 12, 30), datetime.date(2021, 1, 3)].

## Test command

`python tests/runtests.py backends.base.test_operations.SimpleDatabaseOperationTests.test_iso_year_lookup_bounds_for_date_field`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
