# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-11400/a_as_gold/differentiation/django__django-11400--20260908T223120Z--382b74`
- Test: `test_relatedfieldlistfilter_meta_ordering_overrides_manager_ordering`
- Test command: `python tests/runtests.py admin_filters.tests.ListFiltersTests.test_relatedfieldlistfilter_meta_ordering_overrides_manager_ordering`

## Specification gap

When no related ModelAdmin ordering is defined, RelatedFieldListFilter must explicitly apply the related model's Meta.ordering, overriding conflicting ordering introduced by its default manager.

## Input/output contract

Input: Create related authors named A and Z. Their model declares ascending Meta.ordering by name, while its default manager orders names descending. Expose them through a RelatedFieldListFilter without registering a ModelAdmin for the author model.

Expected output: The filter's lookup choices are A followed by Z, as required by Meta.ordering. candidate_a explicitly reapplies Meta.ordering; candidate_b leaves the manager's Z-then-A ordering intact.

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
