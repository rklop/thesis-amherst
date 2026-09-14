# Differentiating-test run: `django__django-12209`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12209:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12209--20260818T094752Z--7a6360`
- Test: `test_raw_save_with_default_primary_key`
- Test command: `python tests/runtests.py serializers.test_natural.NaturalKeySerializerTests.test_raw_save_with_default_primary_key`

## Specification gap

A raw model save must bypass the primary-key-default INSERT optimization regardless of how the primary key was assigned. Its behavior must not depend on detecting an explicit PK during Model.__init__() or assignment through the generic pk property.

## Input/output contract

Input: Create a NaturalPKWithDefault row, construct another instance so its UUID default is generated normally, assign the existing UUID through the model's public id field, change name, and call save_base(raw=True).

Expected output: The existing row is updated to name='updated', no duplicate-key error occurs, and the table still contains exactly one row.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test reveals a genuine specification gap: raw saves (used by serialization) must unconditionally bypass the pk-default INSERT optimization, regardless of how the pk was assigned. Candidate B correctly fixes this by adding `not raw` to the condition. Candidate A fails because its explicit-pk tracking doesn't cover direct attribute assignment (obj.id = value), which is a valid way to set pk in serialization workflows. The test exercises a legitimate entry point (direct field assignment) that represents real usage in loaddata/serialization.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
