# test_partial_repr_uses_current_arguments

- **Instance:** `django__django-14155`
- **Test ID:** `django__django-14155--7dd54695a651da55`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

ResolverMatch.__repr__() should describe the actual partial callback and its current bound arguments, rather than a stale snapshot captured when ResolverMatch was initialized.

## Expected behavior

repr(match) contains repr(callback), including template_name='changed.html'. The supplied gold reads the live partial; the generated candidate reconstructs it from stale copied keywords.

## Test command

`./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_partial_repr_uses_current_arguments`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
