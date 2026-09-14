# test_stale_module_spec

- **Instance:** `django__django-13837`
- **Test ID:** `django__django-13837--00e56d5789551921`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

get_child_arguments() must only reconstruct a `python -m package` invocation when `__main__` module metadata describes the current `sys.argv[0]`. Stale metadata for a different entry point must not override an ordinary script invocation.

## Expected behavior

The child command is `[sys.executable, __file__, 'runserver']`, preserving the current script invocation rather than changing it to `[sys.executable, '-m', 'django', 'runserver']`.

## Test command

`python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_stale_module_spec`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
