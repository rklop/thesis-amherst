# Differentiating-test run: `matplotlib__matplotlib-14623`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-14623:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-14623/b_as_gold/differentiation/matplotlib__matplotlib-14623--20260908T173441Z--e7b7a9`
- Test: `test_log_scale_inverted_limits_with_positive_domain_scalar`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_log_inverted_limits.py`

## Specification gap

For two descending positive log limits, inversion follows from their ordering and the positivity of the smaller endpoint; preserving inversion must not require an additional greater-than-zero comparison on the larger scalar.

## Input/output contract

Input: Call the public tuple form of Axes.set_ylim on a log-scaled y-axis with limits (10, 1). The upper value is a finite float subclass whose greater-than comparison is defined only against values inside the positive log domain.

Expected output: set_ylim completes without error, get_ylim returns (10.0, 1.0), and yaxis_inverted() is true.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
