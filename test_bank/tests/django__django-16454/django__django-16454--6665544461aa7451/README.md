# test_subparser_preserves_programmatic_mode

- **Instance:** `django__django-16454`
- **Test ID:** `django__django-16454--6665544461aa7451`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Subparsers must inherit the parent CommandParser’s exact called_from_command_line value, including None. Otherwise, a custom parser_class with a different default can silently change programmatic error handling into command-line termination.

## Expected behavior

Parsing raises CommandError with 'Error: the following arguments are required: name'. It must not terminate with SystemExit.

## Test command

`./tests/runtests.py user_commands.test_command_parser.CommandParserTests.test_subparser_preserves_programmatic_mode`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
