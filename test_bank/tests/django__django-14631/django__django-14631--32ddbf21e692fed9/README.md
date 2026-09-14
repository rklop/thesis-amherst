# FormsTestCase.test_filefield_initial_callable

- **Instance:** `django__django-14631`
- **Test ID:** `django__django-14631--32ddbf21e692fed9`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

An enabled FileField with no uploaded replacement must reuse the callable initial value cached by its BoundField during cleaning, rather than evaluating the callable again.

## Expected behavior

The form has no errors and cleaned_data['file1'] is "resume.txt", identical to the initial value already exposed by the BoundField.

## Test command

`cd /testbed && python tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_filefield_initial_callable`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
