# test_negated_q_reuses_filtered_relation_join

- **Instance:** `django__django-11265`
- **Test ID:** `django__django-11265--5f11d0e7912806d2`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Predicates combined in one filter() call against the same FilteredRelation alias must apply to the same joined row; a sibling related row must not incorrectly satisfy the negated predicate's exclusion subquery.

## Expected behavior

The queryset contains Alice because the joined “The book by Alice” row satisfies both predicates, regardless of the sibling “The book by Jane C” row.

## Test command

`./tests/runtests.py filtered_relation.tests.FilteredRelationTests.test_negated_q_reuses_filtered_relation_join`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
