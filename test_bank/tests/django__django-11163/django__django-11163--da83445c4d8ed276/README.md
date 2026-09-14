# test_empty_fields_do_not_save_declared_many_to_many_field

- **Instance:** `django__django-11163`
- **Test ID:** `django__django-11163--da83445c4d8ed276`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

An empty ModelForm Meta.fields list means no model fields are selected for persistence, including many-to-many fields that are explicitly declared on the form and therefore appear in cleaned_data.

## Expected behavior

The item's persisted colour relation remains the original colour; the submitted replacement is not saved because colours isn't selected by Meta.fields.

## Test command

`cd /testbed && ./tests/runtests.py model_forms.tests.ModelFormBaseTest.test_empty_fields_do_not_save_declared_many_to_many_field`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
