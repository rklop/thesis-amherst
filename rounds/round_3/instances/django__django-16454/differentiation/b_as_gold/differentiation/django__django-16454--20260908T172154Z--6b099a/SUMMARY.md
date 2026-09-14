# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16454/b_as_gold/differentiation/django__django-16454--20260908T172154Z--6b099a`
- Test: `test_subparser_parser_class_preserves_default`
- Test command: `cd /testbed && python tests/runtests.py user_commands.tests.CommandTests.test_subparser_parser_class_preserves_default`

## Specification gap

When a parent CommandParser leaves called_from_command_line unspecified (None), creating a subparser with a custom CommandParser subclass must preserve that subclass's own default instead of explicitly overriding it with None.

## Input/output contract

Input: Create a CommandParser whose CLI-origin state is unspecified, configure add_subparsers() with a custom CommandParser subclass that defaults to command-line error handling, add a required positional argument to the child parser, and parse only the child command name.

Expected output: Parsing exits with status 2 and writes a human-facing argparse error containing "manage.py command child: error: the following arguments are required: name" to stderr, rather than raising CommandError with a traceback-producing path.

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
