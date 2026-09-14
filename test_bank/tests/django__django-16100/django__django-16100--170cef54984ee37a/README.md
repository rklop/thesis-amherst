# test_list_editable_rolls_back_if_success_message_fails

- **Instance:** `django__django-16100`
- **Test ID:** `django__django-16100--170cef54984ee37a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

A list_editable submission must remain atomic through success reporting: if reporting success raises an exception after the model has been saved, the submitted database changes must be rolled back.

## Expected behavior

The POST raises `RuntimeError("message failure")`, and refreshing the Person from the database shows `alive` is still true.

## Test command

`python tests/runtests.py admin_views.tests.AdminViewListEditable.test_list_editable_rolls_back_if_success_message_fails`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
