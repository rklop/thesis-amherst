# Differentiating-test run: `django__django-15268`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15268:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15268/b_as_gold/differentiation/django__django-15268--20260908T135709Z--ab4f9c`
- Test: `test_later_builtin_operation_replaces_custom_subclass`
- Test command: `cd /testbed && python tests/runtests.py migrations.test_optimizer_subclasses.AlterTogetherSubclassOptimizerTests.test_later_builtin_operation_replaces_custom_subclass`

## Specification gap

Successive AlterFooTogether operations for the same model and option should collapse based on the option they alter, even when the earlier operation is a custom subclass rather than the exact same Python class.

## Input/output contract

Input: Pass MigrationOptimizer an empty CustomAlterUniqueTogether for Book followed by a built-in AlterUniqueTogether that sets Book.unique_together to {('title', 'author')}.

Expected output: The optimizer returns a one-element list containing only the later built-in AlterUniqueTogether operation.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
