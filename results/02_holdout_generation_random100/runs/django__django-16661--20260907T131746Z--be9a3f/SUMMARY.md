# Differentiating-test run: `django__django-16661`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16661:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-16661--20260907T131746Z--be9a3f`
- Test: `test_lookup_allowed_concrete_parent_link`
- Test command: `python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_concrete_parent_link`

## Specification gap

ModelAdmin.lookup_allowed() must treat an explicit concrete-inheritance parent-link traversal as equivalent to direct access to the inherited field. Candidate A incorrectly treats every relational segment as significant, while candidate B preserves this canonicalization.

## Input/output contract

Input: A Referrer model points to a multi-table-inheritance Child. Its ModelAdmin allows the inherited-field filter "child__label". Call lookup_allowed() with the equivalent explicit path "child__parent_ptr__label".

Expected output: lookup_allowed() returns True because Child.label and Child.parent_ptr.label denote the same inherited field path.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The generated test validates that lookup_allowed() correctly handles explicit parent_ptr traversal in multi-table inheritance, which is the semantic inverse of the reported bug (FK-as-PK incorrectly treated as concrete inheritance). Candidate B passes because it explicitly checks model._meta.parents.values() to allow parent link traversal, while Candidate A fails by treating every relational segment as significant without distinguishing concrete inheritance links.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
