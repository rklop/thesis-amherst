# test_missing_slash_append_slash_true_enum_query_string

- **Instance:** `django__django-16612`
- **Test ID:** `django__django-16612--2e3cd6e72ebc2fe8`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

When QUERY_STRING is an enum member containing a string value, AdminSite.catch_all_view() must preserve the member's value in the slash-appending redirect rather than rejecting the value or serializing the enum member's name.

## Expected behavior

A 301 response with Location equal to the changelist URL with its trailing slash followed by "?id=123".

## Test command

`cd /testbed && ./tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_enum_query_string`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
