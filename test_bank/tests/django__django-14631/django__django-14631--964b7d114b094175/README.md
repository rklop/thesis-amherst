# test_filefield_clean_custom_boundfield

- **Instance:** `django__django-14631`
- **Test ID:** `django__django-14631--964b7d114b094175`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The BoundField-based fix applies to disabled-field initial values. Cleaning an enabled FileField with no new upload must continue to obtain its existing value independently of the object returned by the public Field.get_bound_field() customization hook.

## Expected behavior

The form is valid and cleaned_data['file1'] equals 'resume.txt'.

## Test command

`python tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_filefield_clean_custom_boundfield`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
