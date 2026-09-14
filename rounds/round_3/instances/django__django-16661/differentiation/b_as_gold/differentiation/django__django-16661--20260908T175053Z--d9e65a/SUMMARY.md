# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16661/b_as_gold/differentiation/django__django-16661--20260908T175053Z--d9e65a`
- Test: `test_lookup_allowed_foreign_key_target_is_nonrelational`
- Test command: `cd /testbed && python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_is_nonrelational`

## Specification gap

A ForeignKey target component must be treated as an additional relation only when the target field is actually relational. Merely exposing an attribute named path_infos must not change a scalar target lookup into a relational lookup.

## Input/output contract

Input: Define a custom scalar IntegerField primary key with an empty path_infos attribute, point a ForeignKey at it, and call ModelAdmin.lookup_allowed("target__code", "1") with no list_filter. The lookup addresses the related primary-key value already available through the local ForeignKey column.

Expected output: lookup_allowed() returns True. The supplied gold checks field.is_relation and preserves the local-value behavior; the generated candidate mistakes the attribute's presence for relationality and returns False.

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
