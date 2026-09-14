# test_alter_alter_unique_model_with_str_subclass_option_name

- **Instance:** `django__django-15268`
- **Test ID:** `django__django-15268--a7664956ccd879bd`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Repeated AlterFooTogether operations are identified by their concrete operation class and model. A string subtype used for option_name must not prevent two instances of the same concrete operation class from collapsing into the later operation.

## Expected behavior

MigrationOptimizer returns a one-element list containing the later operation, whose target is {("a", "c")}.

## Test command

`python tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_alter_unique_model_with_str_subclass_option_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
