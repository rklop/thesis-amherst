# test_reverse_get_choices_with_meta_ordering_none

- **Instance:** `django__django-11400`
- **Test ID:** `django__django-11400--63659194d21ac6df`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

When reverse-relation choices fall back to the related model's Meta.ordering, None means no default ordering and must be normalized to an empty ordering.

## Expected behavior

Choices for both Bar objects are returned, in either order, without raising TypeError.

## Test command

`cd /testbed && python tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_reverse_get_choices_with_meta_ordering_none`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
