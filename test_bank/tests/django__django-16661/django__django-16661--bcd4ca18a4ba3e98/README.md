# test_lookup_allowed_nontraversable_primary_key_relation

- **Instance:** `django__django-16661`
- **Test ID:** `django__django-16661--bcd4ca18a4ba3e98`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

When a listed relation is followed by its target primary-key field, that terminal key lookup remains authorized even if the key is itself a relation with no further traversal path. It must not be treated as a separate relation requiring its own list_filter entry.

## Expected behavior

ModelAdmin.lookup_allowed() returns True because restaurant__place addresses the target key already represented by the authorized restaurant relation.

## Test command

`python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
