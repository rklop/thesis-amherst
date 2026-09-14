# Differentiating-test run: `django__django-15268`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15268:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15268/b_as_gold/differentiation/django__django-15268--20260908T174113Z--e7285d`
- Test: `test_alter_together_different_options_custom_operation`
- Test command: `cd /testbed && ./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_together_different_options_custom_operation`

## Specification gap

For custom AlterTogetherOptionOperation subclasses, operation class equality must not override option identity. Two same-class operations targeting different together options are independent and must not absorb each other.

## Input/output contract

Input: Pass MigrationOptimizer two instances of one reusable custom operation class for model Book. The first sets unique_together to {('title',)} and the second sets index_together to {('author',)}.

Expected output: The optimized operations expose option_name values ['unique_together', 'index_together'] in that order, showing that both independent changes were retained.

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
