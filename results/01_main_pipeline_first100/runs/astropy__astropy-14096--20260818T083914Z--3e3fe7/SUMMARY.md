# Differentiating-test run: `astropy__astropy-14096`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.astropy_1776_astropy-14096:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/astropy__astropy-14096--20260818T083914Z--3e3fe7`
- Test: `test_subclass_property_preserves_attribute_error_type`
- Test command: `cd /testbed && python -m pytest -q astropy/coordinates/tests/test_sky_coord.py::test_subclass_property_preserves_attribute_error_type`

## Specification gap

SkyCoord subclass properties should preserve not only the nested AttributeError message, but also its public exception subtype when attribute lookup is customized by the subclass.

## Input/output contract

Input: A SkyCoord subclass defines a property that accesses `random_attr`. Its public `__getattribute__` hook raises a custom `MissingAttribute` subtype for that name.

Expected output: Accessing `coord.prop` raises `MissingAttribute`, preserving the nested lookup failure instead of replacing it with a newly constructed base AttributeError.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **low_signal**
- Confidence: 0.7
- Summary: The test reveals a real behavioral difference between candidates but extends beyond the scope of the original issue by testing exception type preservation rather than just error message content. The original issue only complained about misleading error messages (saying 'prop' instead of 'random_attr'), not about preserving custom exception subtypes.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
