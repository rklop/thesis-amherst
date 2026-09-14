# test_missing_slash_preserves_lazy_query_string

- **Instance:** `django__django-16612`
- **Test ID:** `django__django-16612--b22cd76c427ad80c`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Query-string preservation should depend on its serialized value, not require the value stored in request.META to support direct string concatenation. This narrowly covers Django lazy values that resolve to valid query-string text.

## Expected behavior

The response has status 301 and Location "/test_admin/admin/admin_views/article/?id=123". candidate_b serializes the lazy value with %s; candidate_a raises TypeError while concatenating it with a string.

## Test command

`cd /testbed && ./tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_preserves_lazy_query_string`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
