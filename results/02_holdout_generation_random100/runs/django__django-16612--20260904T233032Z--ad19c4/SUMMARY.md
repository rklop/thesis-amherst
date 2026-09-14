# Differentiating-test run: `django__django-16612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-16612--20260904T233032Z--ad19c4`
- Test: `test_missing_slash_append_slash_true_encoded_question_mark_and_query`
- Test command: `cd /testbed && python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_encoded_question_mark_and_query --verbosity 0`

## Specification gap

The slash-appending admin redirect must preserve both query strings and the distinction between encoded path data and URL delimiters. A percent-encoded question mark in an object ID must remain %3F in the redirected path rather than becoming a literal query delimiter.

## Input/output contract

Input: An authenticated staff user requests the slashless admin change URL for object ID "question?mark", encoded in the URL as question%3Fmark, with the query string source=search.

Expected output: A 301 response whose Location is /test_admin/admin/admin_views/article/question%3Fmark/change/?source=search. Candidate B escapes the path through get_full_path(); candidate A emits the decoded question mark literally and therefore produces a semantically different URL.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test reveals a meaningful semantic difference between the two candidates. Candidate A preserves query strings but incorrectly decodes percent-encoded characters in the path (turning %3F into a literal ?), while candidate B correctly preserves the full URL semantics using Django's get_full_path(). This is not merely a test of implementation details but exercises a genuine public URL invariant: object IDs containing encoded delimiters must remain encoded after the redirect. The test is minimal and general, testing URL redirect behavior rather than internal implementation.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
