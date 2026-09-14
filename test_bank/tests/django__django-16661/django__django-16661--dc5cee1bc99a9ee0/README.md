# test_lookup_allowed_non_traversable_target_relation

- **Instance:** `django__django-16661`
- **Test ID:** `django__django-16661--dc5cee1bc99a9ee0`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

A target field should extend lookup_allowed()'s relational path only when the field is itself ORM-traversable. Merely having relational metadata is insufficient.

## Expected behavior

lookup_allowed() returns True. GenericForeignKey is relational but has no traversal path, so the terminal component does not introduce another relation requiring list_filter authorization.

## Test command

`python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_non_traversable_target_relation --verbosity 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
