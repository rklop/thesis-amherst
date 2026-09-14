# test_subparser_explicit_called_from_command_line

- **Instance:** `django__django-16454`
- **Test ID:** `django__django-16454--f8d2d5819765cc71`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A subparser inherits its parent’s command-line error mode only as a default. An explicit called_from_command_line value passed to add_parser() must take precedence.

## Expected behavior

parse_args() raises CommandError with "Error: the following arguments are required: name" instead of printing usage and raising SystemExit.

## Test command

`./tests/runtests.py user_commands.test_command_parser.CommandParserTests.test_subparser_explicit_called_from_command_line`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
