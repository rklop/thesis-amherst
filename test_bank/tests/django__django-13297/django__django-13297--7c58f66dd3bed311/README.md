# test_simple_lazy_object_with_uuid_parameter

- **Instance:** `django__django-13297`
- **Test ID:** `django__django-13297--7c58f66dd3bed311`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

SQLite must accept a SimpleLazyObject as a query parameter by adapting its string representation, even when the wrapped value is not itself a SQLite-native scalar. This covers UUID-valued URL kwargs in addition to the reported slug-string case.

## Expected behavior

The query succeeds and fetchone()[0] is the string '550e8400-e29b-41d4-a716-446655440000'.

## Test command

`cd /testbed && ./tests/runtests.py backends.sqlite.tests.Tests.test_simple_lazy_object_with_uuid_parameter -v 2`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
