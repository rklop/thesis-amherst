# Differentiating-test run: `django__django-15022`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15022:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-15022--20260904T225313Z--4e270b`
- Test: `test_multi_word_search_does_not_combine_q_objects_pairwise`
- Test command: `cd /testbed && python tests/runtests.py admin_changelist.test_search_q_construction.SearchQConstructionTests.test_multi_word_search_does_not_combine_q_objects_pairwise`

## Specification gap

A multi-word admin search should assemble all term predicates as one grouped condition rather than repeatedly combining Q objects, since pairwise construction scales quadratically with search-term arity.

## Input/output contract

Input: A Concert named "Tiny desk concert" belongs to the group "The Hype". ModelAdmin searches both concert and group names for the two-word query "Tiny Hype" using a Q subclass that supports grouped construction but rejects pairwise conjunction.

Expected output: The search returns exactly the created Concert and reports that duplicates aren't possible.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test verifies that multi-word admin search does not combine Q objects pairwise, which is the root cause of the unnecessary JOINs issue. Candidate B passes by collecting term queries and constructing one combined Q object, while candidate A fails because it uses the &= operator which invokes pairwise combination. The test checks implementation approach but also verifies correct search results, making it a valid specification conformance test.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
