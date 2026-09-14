# Differentiating-test run: `astropy__astropy-13579`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.astropy_1776_astropy-13579:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/astropy__astropy-13579--20260818T083634Z--a6c821`
- Test: `test_world_to_pixel_uses_reference_inside_offset_slice`
- Test command: `python -m pytest -q astropy/wcs/wcsapi/wrappers/tests/test_sliced_wcs.py::test_world_to_pixel_uses_reference_inside_offset_slice`

## Specification gap

Reconstructing a sliced-out world coordinate must use a pixel within the retained slice's coordinate domain. The existing regression only uses retained slices starting at zero, so it does not reveal whether nonzero slice offsets are honored.

## Input/output contract

Input: A bounded 3D linear WCS has coupled first and third axes. Pixel z=5 is fixed, while retained x and y ranges start at 10 and 20. The sliced WCS converts sliced pixel (2, 3) to world coordinates and then back to pixels.

Expected output: The public round trip returns sliced pixel coordinates (2.0, 3.0). Candidate A evaluates the forward transform at underlying (0, 0, 5), outside the declared domain, producing NaN for the reconstructed dropped coordinate. Candidate B evaluates at sliced origin (10, 20, 5), which is in bounds.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test verifies a fundamental round-trip invariant (pixel->world->pixel) for SlicedLowLevelWCS with coupled axes and non-zero slice offsets. It exposes a real bug where candidate_a evaluates the dropped world coordinate at the underlying origin (0,0,5), outside the declared pixel domain, producing NaN, while candidate_b correctly evaluates at the sliced origin (10,20,5) which is in bounds. The test is general, uses the public API, and has a clear defensible oracle based on the expected round-trip behavior.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
