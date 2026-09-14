# test_nested_partial_view_name

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--3d750fa5f9ccdd27`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

ResolverMatch must recursively unwrap partial views when deriving the fallback view_name. Unwrapping only one layer can expose "functools.partial" instead of the ultimate view function.

## Expected behavior

The public view_name is "urlpatterns_reverse.views.empty_view", identifying the ultimate underlying view.

## Test command

`cd /testbed && python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_nested_partial_view_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
