# test_lookup_allowed_traversable_primary_key

- **Instance:** `django__django-16661`
- **Test ID:** `django__django-16661--01774e12b2ba14f1`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A configured list_filter path must remain allowed when it crosses a relational primary-key field that provides ORM traversal path information, even if a custom field's is_relation flag is false. Such a traversable primary key must not be collapsed as though it were merely the preceding relation's scalar target field.

## Expected behavior

lookup_allowed("restaurant__place__country", "test_value") returns True because the exact traversable relation path is explicitly configured in list_filter.

## Test command

`cd /testbed && python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_traversable_primary_key`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
