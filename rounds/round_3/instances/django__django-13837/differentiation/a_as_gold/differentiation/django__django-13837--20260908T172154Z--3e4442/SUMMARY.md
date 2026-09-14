# Differentiating-test run: `django__django-13837`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13837:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-13837/a_as_gold/differentiation/django__django-13837--20260908T172154Z--3e4442`
- Test: `test_run_as_non_django_module_without_django_main_file`
- Test command: `python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_run_as_non_django_module_without_django_main_file`

## Specification gap

Module-mode child argument reconstruction must rely on __main__.__spec__.parent and must not require django.__main__.__file__, because Python environments are permitted to omit module __file__ attributes.

## Input/output contract

Input: Set sys.argv to a custom package's __main__.py invocation with runserver, set the executing __main__.__spec__.parent to "custom_package", and remove django.__main__.__file__.

Expected output: get_child_arguments() returns [sys.executable, '-m', 'custom_package', 'runserver'] without raising an exception.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
