# test_alter_alter_unique_model_subclass

- **Instance:** `django__django-15268`
- **Test ID:** `django__django-15268--5e85f611c7783b87`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A subclass of AlterUniqueTogether remains part of the same operation family, so a later subclass instance targeting the same model must supersede an earlier AlterUniqueTogether instance.

## Expected behavior

The optimized sequence contains only the later CustomAlterUniqueTogether operation with {('a', 'c')}.

## Test command

`./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_alter_unique_model_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
