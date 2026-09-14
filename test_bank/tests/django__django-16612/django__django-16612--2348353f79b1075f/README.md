# test_missing_slash_preserves_encoded_question_mark_and_query

- **Instance:** `django__django-16612`
- **Test ID:** `django__django-16612--2348353f79b1075f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

When appending a slash, the redirect must preserve the distinction between percent-encoded path data and the actual query-string delimiter. Reconstructing the URL from the decoded request path can turn an encoded question mark into a query delimiter.

## Expected behavior

A 301 response whose Location is `/test_admin/admin/admin_views/article/%3F/change/?test=1`: the slash is inserted before the query string, `%3F` remains path data, and the query string is preserved.

## Test command

`./tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_preserves_encoded_question_mark_and_query`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
