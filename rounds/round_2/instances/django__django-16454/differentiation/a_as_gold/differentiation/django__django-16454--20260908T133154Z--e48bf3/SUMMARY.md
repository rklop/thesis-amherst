# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-16454/a_as_gold/differentiation/django__django-16454--20260908T133154Z--e48bf3`
- Test: `test_subparser_explicit_called_from_command_line`
- Test command: `./tests/runtests.py user_commands.test_command_parser.CommandParserTests.test_subparser_explicit_called_from_command_line`

## Specification gap

A subparser inherits its parent’s command-line error mode only as a default. An explicit called_from_command_line value passed to add_parser() must take precedence.

## Input/output contract

Input: Create a CLI-mode CommandParser, add a child parser explicitly configured for programmatic use, require a positional name on the child, and parse ["child"] without that name.

Expected output: parse_args() raises CommandError with "Error: the following arguments are required: name" instead of printing usage and raising SystemExit.

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
