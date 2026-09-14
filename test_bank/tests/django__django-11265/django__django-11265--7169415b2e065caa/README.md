# test_exclude_with_outer_field_reference

- **Instance:** `django__django-11265`
- **Test ID:** `django__django-11265--7169415b2e065caa`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When exclude() traverses a FilteredRelation, F() references in the relation's condition must remain correlated with the same outer model row during subquery rewriting.

## Expected behavior

All three authors are returned in primary-key order because none owns a book whose title equals their own name. A title matching a different author's name must not exclude the book's owner.

## Test command

`python tests/runtests.py filtered_relation.tests.FilteredRelationTests.test_exclude_with_outer_field_reference`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
