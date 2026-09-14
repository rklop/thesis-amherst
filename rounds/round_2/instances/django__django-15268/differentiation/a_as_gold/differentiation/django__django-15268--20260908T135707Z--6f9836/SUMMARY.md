# Differentiating-test run: `django__django-15268`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15268:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15268/a_as_gold/differentiation/django__django-15268--20260908T135707Z--6f9836`
- Test: `test_alter_unique_together_custom_subclass`
- Test command: `./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_unique_together_custom_subclass`

## Specification gap

The new optimization may pass through AlterTogether operations for different options, but operations sharing an option must retain the existing class-sensitive reduction semantics. A custom AlterUniqueTogether subclass must not be assumed equivalent to a later built-in AlterUniqueTogether operation.

## Input/output contract

Input: Pass MigrationOptimizer.optimize() a custom AlterUniqueTogether subclass instance followed by a built-in AlterUniqueTogether instance targeting the same model with a different unique_together value.

Expected output: The optimized list contains both original operations in their original order. candidate_a delegates this case to the class-sensitive superclass behavior; candidate_b incorrectly discards the custom subclass operation.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
