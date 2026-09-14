# test_get_choices_preserves_meta_ordering_during_relation_resolution

- **Instance:** `django__django-11400`
- **Test ID:** `django__django-11400--8f4d7696edb8e33c`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

For a forward relational field with no explicit ordering argument, get_choices() must capture the related model's Meta.ordering before resolving the relation's target value field.

## Expected behavior

get_choices(include_blank=False) returns [('a', 'Alpha'), ('z', 'Zulu')], preserving the Meta.ordering in effect when the choice request began.

## Test command

`cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_preserves_meta_ordering_during_relation_resolution`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
