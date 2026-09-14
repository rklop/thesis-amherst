# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16661/a_as_gold/differentiation/django__django-16661--20260908T175003Z--8c163e`
- Test: `test_lookup_allowed_non_traversable_target_relation`
- Test command: `python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_non_traversable_target_relation --verbosity 2`

## Specification gap

A target field should extend lookup_allowed()'s relational path only when the field is itself ORM-traversable. Merely having relational metadata is insufficient.

## Input/output contract

Input: Define an isolated Bookmark relation whose target is TaggedItem.content_object, a GenericForeignKey. Call ModelAdmin.lookup_allowed("tagged_item__content_object", "test_value") with no list_filter entries.

Expected output: lookup_allowed() returns True. GenericForeignKey is relational but has no traversal path, so the terminal component does not introduce another relation requiring list_filter authorization.

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
