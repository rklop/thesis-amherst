# test_alter_unique_together_custom_subclass

- **Instance:** `django__django-15268`
- **Test ID:** `django__django-15268--077c60f0493a2789`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

The new optimization may pass through AlterTogether operations for different options, but operations sharing an option must retain the existing class-sensitive reduction semantics. A custom AlterUniqueTogether subclass must not be assumed equivalent to a later built-in AlterUniqueTogether operation.

## Expected behavior

The optimized list contains both original operations in their original order. candidate_a delegates this case to the class-sensitive superclass behavior; candidate_b incorrectly discards the custom subclass operation.

## Test command

`./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_unique_together_custom_subclass`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
