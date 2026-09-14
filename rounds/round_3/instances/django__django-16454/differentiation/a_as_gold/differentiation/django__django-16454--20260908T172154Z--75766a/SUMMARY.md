# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16454/a_as_gold/differentiation/django__django-16454--20260908T172154Z--75766a`
- Test: `test_subparser_preserves_programmatic_mode`
- Test command: `./tests/runtests.py user_commands.test_command_parser.CommandParserTests.test_subparser_preserves_programmatic_mode`

## Specification gap

Subparsers must inherit the parent CommandParser’s exact called_from_command_line value, including None. Otherwise, a custom parser_class with a different default can silently change programmatic error handling into command-line termination.

## Input/output contract

Input: Create a default CommandParser, add a subparser through the public parser_class option using a CommandParser subclass whose called_from_command_line default is True, require a positional name, and parse only ['create'].

Expected output: Parsing raises CommandError with 'Error: the following arguments are required: name'. It must not terminate with SystemExit.

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
