# test_subparser_custom_parser_class_cli_error

- **Instance:** `django__django-16454`
- **Test ID:** `django__django-16454--2ef363aaf913160e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A custom parser class supplied through argparse's public parser_class hook must retain Django's command-line error behavior when it subclasses CommandParser.

## Expected behavior

Parsing exits with status 2 and writes subcommand usage plus "error: the following arguments are required: name" to stderr. A CommandError must not escape and produce a traceback.

## Test command

`python tests/runtests.py user_commands.tests.CommandTests.test_subparser_custom_parser_class_cli_error`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
