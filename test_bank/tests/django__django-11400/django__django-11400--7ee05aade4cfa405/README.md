# test_get_choices_uses_finalized_related_ordering

- **Instance:** `django__django-11400`
- **Test ID:** `django__django-11400--7ee05aade4cfa405`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When Field.get_choices() is called without an explicit ordering, its choices should follow the related model's finalized Meta.ordering after the relation's choice-value field is resolved. candidate_b snapshots Meta.ordering too early.

## Expected behavior

The choices must match the related model's resulting default queryset order: [(alpha.pk, 'alpha'), (zulu.pk, 'zulu')].

## Test command

`./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_uses_finalized_related_ordering`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
