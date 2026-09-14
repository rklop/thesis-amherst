# test_non_finite_value_custom_error_message

- **Instance:** `django__django-13212`
- **Test ID:** `django__django-13212--f4605f601eb32c72`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

FloatField must make a rejected non-finite numeric value available as %(value)s when rendering a customized invalid error message. The generated candidate omits this validation path.

## Expected behavior

clean() raises ValidationError whose rendered message is '"inf" is not a finite number.'

## Test command

`python tests/runtests.py forms_tests.field_tests.test_floatfield.FloatFieldTest.test_non_finite_value_custom_error_message`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
