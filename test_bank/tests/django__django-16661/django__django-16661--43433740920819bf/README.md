# test_lookup_allowed_foreign_key_target_field

- **Instance:** `django__django-16661`
- **Test ID:** `django__django-16661--43433740920819bf`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A foreign key lookup expressed through its remote target field, such as main_band__id, is equivalent to filtering the local main_band_id column and must remain allowed without being listed in list_filter. candidate_a incorrectly applies relational-lookup validation to this multi-part spelling.

## Expected behavior

lookup_allowed() returns True because main_band__id addresses the foreign key's locally available target value rather than traversing to an additional related field.

## Test command

`cd /testbed && ./tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_field`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
