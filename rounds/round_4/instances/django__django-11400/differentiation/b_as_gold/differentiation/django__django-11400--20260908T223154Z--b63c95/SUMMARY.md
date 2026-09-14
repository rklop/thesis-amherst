# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-11400/b_as_gold/differentiation/django__django-11400--20260908T223154Z--b63c95`
- Test: `test_relatedfieldlistfilter_empty_ordering_uses_model_ordering`
- Test command: `./tests/runtests.py admin_filters.tests.ListFiltersTests.test_relatedfieldlistfilter_empty_ordering_uses_model_ordering`

## Specification gap

An empty list returned by ModelAdmin.get_ordering() means no explicit ordering, just like an empty tuple, and must preserve the related model's Meta.ordering.

## Input/output contract

Input: A RelatedFieldListFilter for Book.employee where Employee has default name ordering and the registered Employee ModelAdmin returns [] from get_ordering(). John Blue was created before Jack Red.

Expected output: The externally visible filter choices are ordered by employee name: Jack Red, then John Blue.

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
