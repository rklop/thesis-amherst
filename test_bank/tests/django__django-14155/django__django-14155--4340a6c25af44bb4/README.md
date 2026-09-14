# test_repr_partial_uses_current_func

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--4340a6c25af44bb4`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

ResolverMatch.__repr__() should describe the partial currently exposed through its public func attribute, rather than a separate partial cached only during construction.

## Expected behavior

The representation contains func=functools.partial(<the repr of absolute_kwargs_view>, ), identifying the replacement partial's underlying function.

## Test command

`python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_uses_current_func`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
