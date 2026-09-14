# test_falsy_exclude_to_model_to_dict

- **Instance:** `django__django-11163`
- **Test ID:** `django__django-11163--9bad9ae265754b83`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The requested change applies presence semantics to `fields`, but does not change the existing contract that a false-valued `exclude` list is an inactive exclusion filter.

## Expected behavior

The result remains `{'id': None, 'name': 'Ada'}` because the false-valued exclusion list is inactive. Candidate A preserves this behavior; candidate B incorrectly excludes `name`.

## Test command

`cd /testbed && ./tests/runtests.py model_forms.tests.ModelFormBaseTest.test_falsy_exclude_to_model_to_dict`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
