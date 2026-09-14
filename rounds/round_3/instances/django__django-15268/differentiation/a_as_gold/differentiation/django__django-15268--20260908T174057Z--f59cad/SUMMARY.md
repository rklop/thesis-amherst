# Differentiating-test run: `django__django-15268`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15268:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15268/a_as_gold/differentiation/django__django-15268--20260908T174057Z--f59cad`
- Test: `test_alter_alter_unique_model_with_str_subclass_option_name`
- Test command: `python tests/runtests.py migrations.test_optimizer.OptimizerTests.test_alter_alter_unique_model_with_str_subclass_option_name`

## Specification gap

Repeated AlterFooTogether operations are identified by their concrete operation class and model. A string subtype used for option_name must not prevent two instances of the same concrete operation class from collapsing into the later operation.

## Input/output contract

Input: Optimize two instances of the same custom AlterUniqueTogether subclass for model Foo. Both option_name values represent the canonical "unique_together" attribute, but carry distinct tags affecting equality between the string-subtype instances. The operations successively target {("a", "b")} and {("a", "c")}.

Expected output: MigrationOptimizer returns a one-element list containing the later operation, whose target is {("a", "c")}.

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
