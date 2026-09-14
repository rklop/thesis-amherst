# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-16661/b_as_gold/differentiation/django__django-16661--20260908T140057Z--7c6ba3`
- Test: `test_lookup_allowed_foreign_key_target_field`
- Test command: `cd /testbed && ./tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_field`

## Specification gap

A foreign key lookup expressed through its remote target field, such as main_band__id, is equivalent to filtering the local main_band_id column and must remain allowed without being listed in list_filter. candidate_a incorrectly applies relational-lookup validation to this multi-part spelling.

## Input/output contract

Input: Create a ModelAdmin for the existing Concert model, leave list_filter empty, and call lookup_allowed("main_band__id", "test_value"). Concert.main_band is a ForeignKey targeting Band.id.

Expected output: lookup_allowed() returns True because main_band__id addresses the foreign key's locally available target value rather than traversing to an additional related field.

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
