# Differentiating-test run: `matplotlib__matplotlib-24026`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-24026:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-24026/b_as_gold/differentiation/matplotlib__matplotlib-24026--20260908T175225Z--9ed9f6`
- Test: `test_stackplot_single_color_alias`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_single_color_alias`

## Specification gap

`stackplot(colors=...)` must accept a single valid color specification directly, including a `CN` alias, and cycle that color across layers without altering the Axes line-color cycler.

## Input/output contract

Input: Call `Axes.stackplot` with two layers and scalar `colors="C2"`, then plot a regular line on the same Axes.

Expected output: Both stacked areas have the RGBA color represented by `C2`, and the subsequent line uses `C0` because stackplot did not consume or replace the Axes line-color cycle.

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
