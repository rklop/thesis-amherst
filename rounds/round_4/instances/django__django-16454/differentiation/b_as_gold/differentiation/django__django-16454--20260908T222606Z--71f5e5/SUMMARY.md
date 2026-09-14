# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-16454/b_as_gold/differentiation/django__django-16454--20260908T222606Z--71f5e5`
- Test: `test_helper_respects_project_line_length`
- Test command: `python tests/runtests.py user_commands.test_command_parser_source`

## Specification gap

New CommandParser subparser support must preserve Django's repository-wide 88-character source-line invariant configured in setup.cfg.

## Input/output contract

Input: Inspect every source line of CommandParser._make_subparser_class(), including indentation, after applying the candidate patch.

Expected output: The longest source line is at most 88 characters. The supplied gold wraps the helper docstring accordingly; the generated candidate contains a 93-character line.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
