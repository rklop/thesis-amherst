# test_alter_together_subclass_different_option

- **Instance:** `django__django-15268`
- **Test ID:** `django__django-15268--d878136b05787e09`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Alter-together operations representing different model options remain independent even when one custom operation class subclasses the other; inheritance must not cause one option update to absorb the other.

## Expected behavior

The two index_together operations collapse to the final one, while the independent unique_together operation is preserved. The result contains AlterUniqueTogether('Foo', {('a', 'b')}) followed by CustomAlterIndexTogether('Foo', {('a', 'c')}).

## Test command

`./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_together_subclass_different_option`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
