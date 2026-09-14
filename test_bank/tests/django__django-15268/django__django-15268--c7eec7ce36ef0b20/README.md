# test_later_builtin_operation_replaces_custom_subclass

- **Instance:** `django__django-15268`
- **Test ID:** `django__django-15268--c7eec7ce36ef0b20`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Successive AlterFooTogether operations for the same model and option should collapse based on the option they alter, even when the earlier operation is a custom subclass rather than the exact same Python class.

## Expected behavior

The optimizer returns a one-element list containing only the later built-in AlterUniqueTogether operation.

## Test command

`cd /testbed && python tests/runtests.py migrations.test_optimizer_subclasses.AlterTogetherSubclassOptimizerTests.test_later_builtin_operation_replaces_custom_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
