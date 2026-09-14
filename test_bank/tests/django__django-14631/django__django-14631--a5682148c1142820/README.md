# Custom BoundField data is used throughout form processing

- **Instance:** `django__django-14631`
- **Test ID:** `django__django-14631--a5682148c1142820`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

BaseForm must treat BoundField.data as the canonical submitted value for both field cleaning and change detection, including when Field.get_bound_field() supplies a custom BoundField.

## Expected behavior

The form is valid, cleaned_data is {"name": "ALICE"}, and changed_data is empty because both operations observe the normalized BoundField value.

## Test command

`python tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_custom_boundfield_data_used_by_form`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
