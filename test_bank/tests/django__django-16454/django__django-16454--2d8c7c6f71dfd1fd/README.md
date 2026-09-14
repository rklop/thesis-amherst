# test_subparser_does_not_inherit_missing_args_message

- **Instance:** `django__django-16454`
- **Test ID:** `django__django-16454--2d8c7c6f71dfd1fd`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A parent CommandParser's missing_args_message applies only to that parser. Subparsers must not inherit it, because an argument-free subparser remains valid when parsed directly.

## Expected behavior

Parsing returns a namespace with no attributes (vars(namespace) == {}) without raising an exception. candidate_b instead raises CommandError using the parent's message.

## Test command

`cd /testbed && python tests/runtests.py user_commands.tests.CommandTests.test_subparser_does_not_inherit_missing_args_message`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
