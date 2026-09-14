# test_list_editable_message_failure_after_commit

- **Instance:** `django__django-16100`
- **Test ID:** `django__django-16100--94a66923f14b8ae0`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The list_editable transaction covers persistence processing, but not the subsequent success notification. If the public ModelAdmin.message_user() hook fails after successful processing, its exception propagates without rolling back the completed edits.

## Expected behavior

The POST raises RuntimeError("message backend failed"), and refreshing the Person from the database still returns gender=2 because the list edit committed before success notification began.

## Test command

`./tests/runtests.py admin_views.tests.AdminViewListEditable.test_list_editable_message_failure_after_commit`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
