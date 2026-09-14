# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11848--20260818T093841Z--138d45`
- Test: `test_parsing_rfc850_at_next_century_boundary`
- Test command: `python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_at_next_century_boundary`

## Specification gap

Two-digit RFC 850 years must be interpreted relative to the current century, including after a century rollover. A year exactly 50 years in the future must remain in the future because only dates more than 50 years ahead are rolled back.

## Input/output contract

Input: Freeze the current UTC time at 2100-11-06 08:49:37 and parse the RFC 850 date "Sunday, 06-Nov-50 08:49:37 GMT".

Expected output: parse_http_date() returns an epoch timestamp that converts to 2150-11-06 08:49:37 UTC. Candidate A instead resolves the year against a hard-coded 2000 base and produces 2050.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test correctly exercises the RFC 7231 specification for RFC 850 two-digit year parsing. It validates that the year calculation must be relative to the current century (not hard-coded to 2000), specifically testing the 50-year boundary at a century rollover (year 2100). Candidate B correctly computes the current century and applies the >50 year rule, while Candidate A incorrectly adds 2000 as a base, failing when the current year is 2100. The test winner (candidate_b) is specification-conformant.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
