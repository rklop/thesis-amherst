# test_wrapped_partial_view_name

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--d6fa57f49ec8643e`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Unwrapping a partial must preserve the public __module__/__name__ metadata of an inner callable decorated with functools.update_wrapper(); it must not recursively unwrap that decorated partial to its internal dispatch helper.

## Expected behavior

ResolverMatch.view_name equals the public view's module-qualified name. candidate_a unwraps only the outer partial and preserves that identity; candidate_b recursively unwraps to dispatch and fails the assertion.

## Test command

`./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_wrapped_partial_view_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
