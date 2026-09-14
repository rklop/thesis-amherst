# Differentiating-test run: `matplotlib__matplotlib-14623`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-14623:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/matplotlib__matplotlib-14623/a_as_gold/differentiation/matplotlib__matplotlib-14623--20260908T135012Z--d5a92c`
- Test: `test_view_limits_reversed_with_nonpositive_endpoint`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint`

## Specification gap

Reversed log limits preserve inversion only when both endpoints are positive. If one endpoint is non-positive and must be replaced, repairing it must not accidentally create an inverted interval.

## Input/output contract

Input: Plot x data with minimum-positive value 10, use a logarithmic x-axis, then ask its LogLocator for view limits from the reversed boundary pair (1, 0).

Expected output: The invalid zero endpoint is replaced by the axis minimum-positive value, producing the ordered limits (1, 10).

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
