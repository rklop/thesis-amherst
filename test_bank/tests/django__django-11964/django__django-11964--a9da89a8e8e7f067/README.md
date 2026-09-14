# test_str

- **Instance:** `django__django-11964`
- **Test ID:** `django__django-11964--a9da89a8e8e7f067`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The value-like string conversion required by the issue applies to the public Choices abstraction, including documented custom concrete types, not only TextChoices and IntegerChoices.

## Expected behavior

str(MoonLandings.APOLLO_11) returns "1969-07-20", the string representation of its underlying date value.

## Test command

`cd /testbed && ./tests/runtests.py model_enums.tests.CustomChoicesTests.test_str`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
