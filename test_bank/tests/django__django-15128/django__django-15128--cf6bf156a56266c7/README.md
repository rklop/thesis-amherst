# test_or_with_multichar_alias_prefix

- **Instance:** `django__django-15128`
- **Test ID:** `django__django-15128--cf6bf156a56266c7`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

QuerySet combination must avoid alias collisions when Django uses a multi-character alias prefix, not only its initial one-character prefix.

## Expected behavior

The combined QuerySet evaluates without an alias-generation exception and returns exactly the two created BaseUser primary keys.

## Test command

`cd /testbed && ./tests/runtests.py queries.tests.QuerySetBitwiseOperationTests.test_or_with_multichar_alias_prefix`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
