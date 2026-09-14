# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-16661/a_as_gold/differentiation/django__django-16661--20260908T223839Z--5a96fe`
- Test: `test_lookup_allowed_nontraversable_primary_key_relation`
- Test command: `cd /testbed && python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation`

## Specification gap

A non-traversable relation used as a related model's primary key remains the preceding ForeignKey's target field. Looking it up is equivalent to using the ForeignKey's stored target value and must not be classified as an additional relationship requiring list_filter authorization.

## Input/output contract

Input: Call ModelAdmin.lookup_allowed("restaurant__place", "test_value") for Waiter.restaurant pointing to Restaurant, whose primary key is a OneToOneField named place with path_infos=None.

Expected output: lookup_allowed() returns True because place is the terminal target field of restaurant, not a second traversed relationship.

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
