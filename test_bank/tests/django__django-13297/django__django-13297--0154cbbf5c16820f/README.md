# test_conform_database_adapter_contract

- **Instance:** `django__django-13297`
- **Test ID:** `django__django-13297--0154cbbf5c16820f`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

LazyObject.__conform__ is specifically intended to implement the PEP 246 database-adapter protocol so lazy values can be query parameters, rather than being an undocumented generic coercion hook.

## Expected behavior

The comments identify PEP 246 and database-adapter support. candidate_b supplies this contract annotation; candidate_a does not.

## Test command

`cd /testbed && python tests/runtests.py utils_tests.test_lazyobject.LazyObjectTestCase.test_conform_database_adapter_contract`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
