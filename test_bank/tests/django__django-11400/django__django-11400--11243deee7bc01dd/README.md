# test_get_choices_falls_back_to_falsy_nonempty_model_ordering

- **Instance:** `django__django-11400`
- **Test ID:** `django__django-11400--11243deee7bc01dd`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

A related model's Meta.ordering is an ordering sequence, not merely a boolean flag. A valid tuple subclass may be false-valued while still containing ordering terms; get_choices() must preserve and apply those terms when falling back to Meta.ordering.

## Expected behavior

The choices are ordered by '-a': Foo 'b' appears before Foo 'a'.

## Test command

`cd /testbed && python tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_falls_back_to_falsy_nonempty_model_ordering`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
