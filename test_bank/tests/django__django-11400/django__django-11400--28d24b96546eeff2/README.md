# test_get_choices_model_ordering_uses_resolved_related_model

- **Instance:** `django__django-11400`
- **Test ID:** `django__django-11400--28d24b96546eeff2`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Fallback ordering must come from the same resolved related model whose objects populate the choices. candidate_a performs a second target-model lookup, while candidate_b consistently reuses the initially resolved model.

## Expected behavior

Choices contain foo2 followed by foo1, matching descending Foo.a ordering. candidate_a instead applies the later ascending ordering and returns the opposite order.

## Test command

`cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_model_ordering_uses_resolved_related_model`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
