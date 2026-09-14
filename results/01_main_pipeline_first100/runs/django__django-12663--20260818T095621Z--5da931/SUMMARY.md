# Differentiating-test run: `django__django-12663`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12663:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12663--20260818T095621Z--5da931`
- Test: `test_subquery_annotation_filter_lazy_to_field`
- Test command: `python tests/runtests.py many_to_one.tests.ManyToOneTests.test_subquery_annotation_filter_lazy_to_field`

## Specification gap

A scalar subquery selecting a ForeignKey must retain the relation’s configured target-field semantics when compared with a lazy model instance. The value must come from the ForeignKey’s `to_field`, which is not necessarily the related object’s primary key.

## Input/output contract

Input: Create a Parent with primary key 7 and unique name `parent-key`, plus a ToFieldChild whose ForeignKey targets Parent.name. Annotate the child with a correlated scalar subquery selecting that ForeignKey, then retrieve it by comparing the annotation to a SimpleLazyObject wrapping the Parent.

Expected output: The annotated lookup returns the created ToFieldChild. The comparison must use `parent-key`, not the Parent primary key `7`, and must not raise a lazy-object conversion error.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test reveals a genuine semantic gap: when filtering by a lazy model instance through a subquery annotation that selects a ForeignKey field, the comparison must use the ForeignKey's configured target field (to_field), not blindly convert to pk. Candidate B correctly preserves the ForeignKey metadata through the subquery, while Candidate A's approach of always using .pk fails for non-pk target fields. The test uses the to_field relationship correctly to demonstrate this distinction.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
