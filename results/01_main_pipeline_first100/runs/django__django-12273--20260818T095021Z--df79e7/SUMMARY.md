# Differentiating-test run: `django__django-12273`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12273:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12273--20260818T095021Z--df79e7`
- Test: `test_save_with_explicit_new_pk`
- Test command: `./tests/runtests.py model_inheritance_regress.tests.ModelInheritanceTest.test_save_with_explicit_new_pk`

## Specification gap

Assigning a valid non-None value through a child model's public pk property must propagate that value to the target primary-key field of a non-primary parent link. The existing tests cover only None and manually clear link fields, so they don't distinguish updating the parent's target field from incorrectly updating the child link column.

## Input/output contract

Input: Create a Profile whose own primary key is distinct from its inherited User identity, assign an unused integer to profile.pk, change its username, and save it.

Expected output: The save succeeds and creates a second Profile/User pair. The original profile still has username 'john'; the copied profile has the explicit new primary key, username 'bill', and a User parent with that same new identity.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test exercises a legitimate public API contract: setting pk on a child model instance should propagate the value to parent link fields to enable creating a new row on save. The test's oracle correctly verifies both the original and new records exist, and candidate_b's passing indicates it implements the correct semantics by propagating the pk to the parent id field rather than just the child link column.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
