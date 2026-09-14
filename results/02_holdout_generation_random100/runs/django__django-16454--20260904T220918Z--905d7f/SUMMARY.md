# Differentiating-test run: `django__django-16454`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16454:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-16454--20260904T220918Z--905d7f`
- Test: `test_subparser_custom_parser_class`
- Test command: `python tests/runtests.py user_commands.tests.CommandParserTests.test_subparser_custom_parser_class`

## Specification gap

The public `parser_class` extension point must instantiate the supplied `CommandParser` subclass directly while propagating `called_from_command_line`; it must not require that supplied class to support further subclassing.

## Input/output contract

Input: A command-line `CommandParser` creates a `subcommand` parser using a custom `CommandParser` class that prohibits further subclassing. The child has a required positional `name`, and the root parser receives only `subcommand`.

Expected output: Parser construction succeeds. Parsing exits with status 2 and stderr contains `command subcommand: error: the following arguments are required: name`.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.85
- Summary: The test validates that the public `parser_class` extension point in `add_subparsers()` correctly propagates `called_from_command_line` to subparsers while allowing users to supply custom CommandParser subclasses. Candidate B passes by using `functools.partial` to instantiate the exact class provided, while candidate A fails because it dynamically subclasses the custom parser, violating a `__init_subclass__` restriction. This is a legitimate edge case of the public argparse API that the fix must handle. The test checks both parser construction success and proper error formatting (the core issue), making it a meaningful differentiation.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
