# test_repr_partial_subclass

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--11b9bfc12e5ca843`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

ResolverMatch must expose the underlying callable and pre-bound arguments for any functools.partial instance, including subclasses whose own repr is opaque.

## Expected behavior

repr(ResolverMatch(...)) contains 'empty_view', 'preset', and "template_name='template.html'" despite the partial subclass hiding those details in its own repr.

## Test command

`cd /testbed && ./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
