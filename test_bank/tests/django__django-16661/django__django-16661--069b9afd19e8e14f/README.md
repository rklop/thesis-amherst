# test_lookup_allowed_relation_primary_key_without_path_info

- **Instance:** `django__django-16661`
- **Test ID:** `django__django-16661--069b9afd19e8e14f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A relationship used as a related model's primary key must remain a relational lookup for admin authorization even when that custom relationship field provides no further traversal path. It must not be collapsed into a scalar local-column lookup.

## Expected behavior

lookup_allowed() returns False because restaurant__place is an unlisted multi-hop relational lookup.

## Test command

`python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_relation_primary_key_without_path_info`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
