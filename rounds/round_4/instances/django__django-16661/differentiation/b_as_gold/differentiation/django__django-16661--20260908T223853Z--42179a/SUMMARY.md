# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-16661/b_as_gold/differentiation/django__django-16661--20260908T223853Z--42179a`
- Test: `test_lookup_allowed_foreign_key_target_non_relation`
- Test command: `cd /testbed && ./tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_non_relation`

## Specification gap

The foreign-key target shortcut applies based on field type: a scalar target remains a locally available value even if a custom field happens to expose relation-style path metadata. Only an actual relational target should extend the traversed lookup path.

## Input/output contract

Input: Define a custom CharField with an empty path_infos attribute, use it as the unique to_field target of Employee.department, and call ModelAdmin.lookup_allowed("department__code", "test_value") without configuring list_filter.

Expected output: lookup_allowed() returns True because department__code resolves to the scalar value already stored by the ForeignKey; the incidental path_infos attribute does not make the target field a relation.

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
