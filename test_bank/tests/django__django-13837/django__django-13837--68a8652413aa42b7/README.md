# test_run_as_module_without_main_file

- **Instance:** `django__django-13837`
- **Test ID:** `django__django-13837--68a8652413aa42b7`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A nonempty `__main__.__spec__.parent` is sufficient to identify a `python -m package` launch. Detection must not additionally require `__main__.__file__`, `__spec__.origin`, or an entry-point path comparison.

## Expected behavior

`get_child_arguments()` returns `[sys.executable, '-m', 'custom_cli', 'runserver']`, preserving the package-based invocation for autoreload.

## Test command

`python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_run_as_module_without_main_file`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
