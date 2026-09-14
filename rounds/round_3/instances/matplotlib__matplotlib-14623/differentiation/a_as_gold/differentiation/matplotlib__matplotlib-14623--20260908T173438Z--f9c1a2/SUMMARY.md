# Differentiating-test run: `matplotlib__matplotlib-14623`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-14623:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-14623/a_as_gold/differentiation/matplotlib__matplotlib-14623--20260908T173438Z--f9c1a2`
- Test: `test_log_inverted_near_singular_limits_preserve_direction`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_log_inverted_limits_regression.py::test_log_inverted_near_singular_limits_preserve_direction`

## Specification gap

Explicitly descending positive log limits must remain descending even when the locator regards the endpoints as singular and automatically expands them.

## Input/output contract

Input: Set a log-scaled x-axis to descending scalar limits 10.1 and 10.0 using a float subtype whose equality reflects whole-unit precision, so the endpoints are ordered as descending but trigger the public singular-limit expansion path.

Expected output: set_xlim emits its singular-limit warning, and the resulting x-axis remains inverted (its left limit is greater than its right limit).

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
