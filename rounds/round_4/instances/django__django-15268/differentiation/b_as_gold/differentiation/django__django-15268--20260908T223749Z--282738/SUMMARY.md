# Differentiating-test run: `django__django-15268`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15268:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-15268/b_as_gold/differentiation/django__django-15268--20260908T223749Z--282738`
- Test: `test_alter_together_reduction_uses_option_name`
- Test command: `cd /testbed && ./tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_together_reduction_uses_option_name`

## Specification gap

Alter-together operations supersede one another based on the model option they alter, not merely their Python inheritance relationship.

## Input/output contract

Input: Optimize an AlterUniqueTogether for Book followed by a subclass that instead targets index_together on the same model.

Expected output: The optimizer returns both independent changes in order: unique_together={('title',)} and index_together={('author',)}. candidate_a incorrectly drops the unique_together change.

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
