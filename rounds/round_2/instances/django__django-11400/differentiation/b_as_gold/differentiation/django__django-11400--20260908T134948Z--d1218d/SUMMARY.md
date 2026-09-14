# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11400/b_as_gold/differentiation/django__django-11400--20260908T134948Z--d1218d`
- Test: `test_reverse_get_choices_with_meta_ordering_none`
- Test command: `cd /testbed && python tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_reverse_get_choices_with_meta_ordering_none`

## Specification gap

When reverse-relation choices fall back to the related model's Meta.ordering, None means no default ordering and must be normalized to an empty ordering.

## Input/output contract

Input: Temporarily set Bar._meta.ordering to None and call get_choices(include_blank=False) through Bar.a's reverse relationship without an explicit ordering.

Expected output: Choices for both Bar objects are returned, in either order, without raising TypeError.

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
