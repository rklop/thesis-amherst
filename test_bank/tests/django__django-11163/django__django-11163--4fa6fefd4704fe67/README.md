# test_falsey_exclude_to_model_to_dict

- **Instance:** `django__django-11163`
- **Test ID:** `django__django-11163--4fa6fefd4704fe67`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

For model_to_dict(), None is the sentinel meaning that exclude wasn't supplied. Any supplied list-like exclude collection must be honored according to membership, even if the collection has a false Boolean value.

## Expected behavior

model_to_dict() returns {'id': None}; the explicitly excluded name field is absent while the unexcluded id remains.

## Test command

`python tests/runtests.py model_forms.tests.ModelFormBaseTest.test_falsey_exclude_to_model_to_dict`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
