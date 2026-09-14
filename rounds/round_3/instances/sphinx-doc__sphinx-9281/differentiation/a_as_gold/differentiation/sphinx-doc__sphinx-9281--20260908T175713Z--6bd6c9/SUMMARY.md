# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/a_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T175713Z--6bd6c9`
- Test: `test_stringify_signature_enum_with_overridden_name`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_inspect.py::test_stringify_signature_enum_with_overridden_name`

## Specification gap

An Enum default must be rendered from its declared symbolic member identity, not from the member's overridable `name` attribute. Otherwise a valid Enum subclass can produce a misleading signature.

## Input/output contract

Input: A function whose default argument is `CustomNameEnum.VALUE`, where the Enum overrides `name` to return `'overridden'`. The test exercises signature stringification, matching the issue's observable behavior.

Expected output: The signature is exactly `(value=CustomNameEnum.VALUE)`. candidate_a derives the declared member identifier and should pass; candidate_b accesses the overridden `name` property and produces `(value=CustomNameEnum.overridden)`.

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
