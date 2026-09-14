# test_missing_slash_append_slash_true_query_string_subclass

- **Instance:** `django__django-16612`
- **Test ID:** `django__django-16612--1fd7da2f6697c84f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

AdminSite.catch_all_view() must preserve the value of QUERY_STRING as string data, including for legitimate str subtypes whose display representation differs from their stored value.

## Expected behavior

A 301 response with Location equal to the slash-appended admin URL followed by "?id=123".

## Test command

`cd /testbed && python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_query_string_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
