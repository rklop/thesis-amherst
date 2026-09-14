# Differentiating-test run: `pydata__xarray-6744`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.pydata_1776_xarray-6744:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/pydata__xarray-6744/a_as_gold/differentiation/pydata__xarray-6744--20260908T224157Z--41799e`
- Test: `test_centered_rolling_iter_dimension_is_stable`
- Test command: `python -m pytest -q xarray/tests/test_rolling.py::TestDataArrayRolling::test_centered_rolling_iter_dimension_is_stable`

## Specification gap

Once manual iteration starts, its rolling dimension should remain the dimension configured at iterator startup. The gold patch captures that dimension once; the generated candidate re-reads mutable metadata between yields and can retarget an active iterator.

## Input/output contract

Input: Create a 3-by-4 DataArray, start centered rolling iteration along x with window size 2, consume one window, change the rolling object's dimension metadata to y, then request the second window.

Expected output: The active iterator still yields the centered x-window containing the first two x rows, identical to da.isel(x=slice(0, 2)).

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
