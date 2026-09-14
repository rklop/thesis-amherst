# test_template_params_can_be_used_in_query

- **Instance:** `django__django-13297`
- **Test ID:** `django__django-13297--db341958c72d20c4`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A URL kwarg exposed in TemplateView's final context must remain usable as an ORM filter value, not merely arrive unwrapped inside an overridden get_context_data().

## Expected behavior

The ORM query completes without a SQLite parameter-binding error and reports that the matching Author exists.

## Test command

`./tests/runtests.py generic_views.test_base.DeprecationTests.test_template_params_can_be_used_in_query`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
