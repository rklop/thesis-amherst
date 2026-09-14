# test_explicit_m2m_field_with_empty_meta_fields_is_saved

- **Instance:** `django__django-11163`
- **Test ID:** `django__django-11163--82bcc0886493a775`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

An empty fields list means model_to_dict() should return no fields, but it must not be generalized to suppress explicitly declared ModelForm fields. A declaratively defined many-to-many field remains part of the form and its cleaned value must be saved even when Meta.fields is empty.

## Expected behavior

The form is valid and form.save() persists the submitted Category in the Article's many-to-many categories relation. candidate_a retains this behavior; candidate_b skips the relation because of its additional _save_m2m() change.

## Test command

`cd /testbed && python tests/runtests.py model_forms.tests.ModelFormBaseTest.test_explicit_m2m_field_with_empty_meta_fields_is_saved`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
