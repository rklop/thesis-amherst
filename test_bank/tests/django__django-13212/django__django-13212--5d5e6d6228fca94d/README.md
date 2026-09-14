# test_decimal_validator_non_finite_value_error_code

- **Instance:** `django__django-13212`
- **Test ID:** `django__django-13212--5d5e6d6228fca94d`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Adding the rejected value to a validator's ValidationError should enrich its params without changing pre-existing error metadata. In particular, DecimalValidator's non-finite-number error must retain its existing None error code.

## Expected behavior

It raises ValidationError whose rendered message is '"NaN" is not a number.', whose params['value'] is the supplied Decimal object, and whose code remains None.

## Test command

`python tests/runtests.py validators.tests.TestValidators.test_decimal_validator_non_finite_value_error_code`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
