# Differentiating-test run: `matplotlib__matplotlib-24026`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-24026:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/matplotlib__matplotlib-24026/a_as_gold/differentiation/matplotlib__matplotlib-24026--20260908T140104Z--8886f8`
- Test: `test_stackplot_color_iterator_does_not_advance_prop_cycle`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_color_iterator_does_not_advance_prop_cycle`

## Specification gap

The explicit `colors` input should continue to accept one-shot iterables, including when they contain `CN` aliases. Normalizing those colors must not alter or advance the Axes property cycle.

## Input/output contract

Input: Create an Axes with the deterministic cycle red, green, blue. Draw two stacked series using `colors=iter(["C1", "C2"])`, then draw an ordinary line on the same Axes.

Expected output: Stackplot completes without error; its two returned areas are green and blue, respectively, while the subsequently plotted line is red (C0), showing that stackplot did not consume or replace the Axes cycle.

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
