# test_repr_partial_callable_object

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--5066f5b63b0275cd`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A partial view must preserve ResolverMatch's existing support for callable objects, even when the underlying callable has no function-style __name__ attribute.

## Expected behavior

Construction succeeds and repr(match) is exactly "ResolverMatch(func=functools.partial(callable_view, setting='value'), args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)".

## Test command

`cd /testbed && python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_callable_object`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
