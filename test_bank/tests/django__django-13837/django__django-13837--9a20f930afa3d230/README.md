# test_main_module_is_resolved_after_django_main

- **Instance:** `django__django-13837`
- **Test ID:** `django__django-13837--9a20f930afa3d230`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

get_child_arguments() must inspect the currently registered top-level __main__ module after Django's CLI module has been imported, rather than retaining a module object captured before that import.

## Expected behavior

The returned child command is [sys.executable, '-m', 'pkg_other_than_django', 'runserver'], reflecting the active entry-point module.

## Test command

`python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_main_module_is_resolved_after_django_main`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
