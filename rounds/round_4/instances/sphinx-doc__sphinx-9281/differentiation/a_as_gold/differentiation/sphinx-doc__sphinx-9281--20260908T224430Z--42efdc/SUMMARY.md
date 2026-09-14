# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/a_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T224430Z--42efdc`
- Test: `test_signature_with_mapping_enum_default`
- Test command: `python -m pytest -q tests/test_util_inspect.py::test_signature_with_mapping_enum_default`

## Specification gap

Enum formatting takes precedence when an Enum member is also a container instance: function defaults must retain the symbolic `EnumClass.Member` representation rather than expose the underlying container value.

## Input/output contract

Input: A function whose default argument is `MappingEnum.DEFAULT`, where `MappingEnum` inherits from both `dict` and `enum.Enum` and the member's value is `{'key': 'value'}`.

Expected output: Stringifying the function signature returns exactly `(value=MappingEnum.DEFAULT)`.

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
