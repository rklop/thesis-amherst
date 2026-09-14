# test_subparser_missing_args_message

- **Instance:** `django__django-16454`
- **Test ID:** `django__django-16454--716ea25eab24e0c7`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

CommandParser subparsers must inherit the parent parser's custom missing_args_message, not only its command-line invocation context.

## Expected behavior

Parsing raises CommandError with the externally visible message "Error: Custom error message." rather than argparse's generic missing-required-argument message.

## Test command

`python tests/runtests.py user_commands.tests.CommandTests.test_subparser_missing_args_message`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
