# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-16661/a_as_gold/differentiation/django__django-16661--20260908T135939Z--a679de`
- Test: `test_lookup_allowed_nontraversable_primary_key_relation`
- Test command: `python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation`

## Specification gap

When a listed relation is followed by its target primary-key field, that terminal key lookup remains authorized even if the key is itself a relation with no further traversal path. It must not be treated as a separate relation requiring its own list_filter entry.

## Input/output contract

Input: Define Restaurant.place as a non-traversable OneToOneField primary key, reference Restaurant from Waiter.restaurant, configure list_filter = ["restaurant"], and call lookup_allowed("restaurant__place", "test_value").

Expected output: ModelAdmin.lookup_allowed() returns True because restaurant__place addresses the target key already represented by the authorized restaurant relation.

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
