# Differentiating-test run: `django__django-15814`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15814:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15814--20260904T223635Z--81fd56`
- Test: `test_select_related_only_proxy_inherited_field`
- Test command: `cd /testbed && python tests/runtests.py proxy_models.tests.ProxyModelTests.test_select_related_only_proxy_inherited_field`

## Specification gap

QuerySet.only() must preserve deferred-field semantics when select_related() traverses a proxy model, including when the requested field belongs to an inherited parent model. Avoiding the crash by changing only the metadata lookup causes candidate A to eagerly load the proxy's unrequested concrete-model fields.

## Input/output contract

Input: Create a ProxyTrackerUser with inherited field name='Joe Doe' and local field status='active', assign it to an Issue, then retrieve the Issue using select_related('assignee').only('assignee__name').

Expected output: The related proxy assignee is populated and its requested name equals 'Joe Doe', while the unrequested status field is reported as deferred.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The generated test exposes a meaningful semantic difference between the candidates. While both candidate_a and candidate_b fix the original crash, candidate_a breaks .only() deferral semantics by over-fetching fields (status is not deferred), whereas candidate_b correctly preserves deferred field behavior. The test reveals that candidate_a only fixes the metadata lookup but fails to properly defer non-requested fields, making candidate_b the specification-conformant fix.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
