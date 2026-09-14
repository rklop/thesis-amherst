# test_run_as_non_django_module_without_django_main_file

- **Instance:** `django__django-13837`
- **Test ID:** `django__django-13837--c058a9c61b71162d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Module-mode child argument reconstruction must rely on __main__.__spec__.parent and must not require django.__main__.__file__, because Python environments are permitted to omit module __file__ attributes.

## Expected behavior

get_child_arguments() returns [sys.executable, '-m', 'custom_package', 'runserver'] without raising an exception.

## Test command

`python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_run_as_non_django_module_without_django_main_file`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
