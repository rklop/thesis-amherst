# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-25479/b_as_gold/differentiation/matplotlib__matplotlib-25479--20260908T174722Z--d18fe8`
- Test: `test_colormap_registry_lookup_uses_mapping_key`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_colors.py::test_colormap_registry_lookup_uses_mapping_key`

## Specification gap

A ColormapRegistry lookup must return a colormap whose name is the registry key used for that lookup, even for entries supplied when the registry is created rather than added later via register().

## Input/output contract

Input: Create a ListedColormap named "intrinsic", place it in a ColormapRegistry under the distinct key "registered_alias", and retrieve it through the registry's documented mapping interface.

Expected output: The retrieved colormap's publicly observable name is "registered_alias", matching the lookup key rather than the original colormap name.

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
