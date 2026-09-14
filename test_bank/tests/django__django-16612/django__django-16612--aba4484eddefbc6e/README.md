# test_missing_slash_preserves_bytes_query_string

- **Instance:** `django__django-16612`
- **Test ID:** `django__django-16612--aba4484eddefbc6e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Query-string preservation must normalize UTF-8 bytes as URI data before composing the redirect, rather than interpolating Python's bytes representation into the URL.

## Expected behavior

A 301 response with Location `/test_admin/admin/admin_views/article/?q=caf%C3%A9`.

## Test command

`python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_preserves_bytes_query_string`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
