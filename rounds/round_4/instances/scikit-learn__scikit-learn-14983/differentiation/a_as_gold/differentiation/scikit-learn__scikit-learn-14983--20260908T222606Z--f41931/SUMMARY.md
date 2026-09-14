# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/a_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T222606Z--f41931`
- Test: `test_repeated_cv_repr_with_read_only_splitter_parameter`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_cv_repr_with_read_only_splitter_parameter`

## Specification gap

Generating a repeated splitter's representation must not require copying wrapped-splitter parameters onto the repeated wrapper. Such parameters may already be exposed through read-only attributes.

## Input/output contract

Input: Construct a repeated splitter with read_only='sentinel', where read_only is exposed by a getter-only property and forwarded to the wrapped splitter.

Expected output: Construction succeeds and repr returns exactly "RepeatedReadOnlySplit(read_only='sentinel')".

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
