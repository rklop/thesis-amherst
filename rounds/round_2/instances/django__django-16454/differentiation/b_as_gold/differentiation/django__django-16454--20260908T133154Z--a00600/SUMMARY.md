# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-16454/b_as_gold/differentiation/django__django-16454--20260908T133154Z--a00600`
- Test: `test_subparser_custom_parser_class_cli_error`
- Test command: `python tests/runtests.py user_commands.tests.CommandTests.test_subparser_custom_parser_class_cli_error`

## Specification gap

A custom parser class supplied through argparse's public parser_class hook must retain Django's command-line error behavior when it subclasses CommandParser.

## Input/output contract

Input: Run a BaseCommand from argv whose "create" subparser uses a custom CommandParser subclass and requires a positional "name", but invoke it with only "create".

Expected output: Parsing exits with status 2 and writes subcommand usage plus "error: the following arguments are required: name" to stderr. A CommandError must not escape and produce a traceback.

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
