# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-15127/b_as_gold/differentiation/django__django-15127--20260908T224703Z--502520`
- Test: `test_imported_level_tags_track_override_settings`
- Test command: `python tests/runtests.py messages_tests.tests.MessageTests.test_imported_level_tags_track_override_settings`

## Specification gap

An already-imported LEVEL_TAGS mapping must reflect temporary MESSAGE_TAGS overrides, not only new lookups through the storage.base module global.

## Input/output contract

Input: Directly import LEVEL_TAGS, then activate override_settings(MESSAGE_TAGS={29: 'custom'}) and query level 29 through that pre-existing reference.

Expected output: LEVEL_TAGS.get(29) returns 'custom' while the override is active.

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
