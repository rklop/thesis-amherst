# Differentiating-test run: `django__django-15127`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15127:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15127/a_as_gold/differentiation/django__django-15127--20260908T141346Z--f41a8d`
- Test: `test_imported_level_tags_track_override`
- Test command: `cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_imported_level_tags_track_override`

## Specification gap

A LEVEL_TAGS reference imported before override_settings() should expose the active MESSAGE_TAGS values. Rebinding only the storage module attribute leaves such references stale.

## Input/output contract

Input: Import LEVEL_TAGS, then override MESSAGE_TAGS so the INFO level maps to 'custom-info', and access INFO through the imported mapping.

Expected output: LEVEL_TAGS[constants.INFO] returns 'custom-info' while the override is active.

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
