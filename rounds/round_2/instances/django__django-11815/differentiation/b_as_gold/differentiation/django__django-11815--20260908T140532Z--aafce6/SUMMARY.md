# Differentiating-test run: `django__django-11815`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11815:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11815/b_as_gold/differentiation/django__django-11815--20260908T140532Z--aafce6`
- Test: `test_named_integer_regex_flag`
- Test command: `python tests/runtests.py migrations.test_regex_flag_serialization.RegexFlagSerializationTests.test_named_integer_regex_flag`

## Specification gap

Compiled-regex serialization should preserve the symbolic name of an int-compatible regex flag when that name remains available after flag normalization, extending the issue's name-over-value rule to this nested enum entry point.

## Input/output contract

Input: A Django RegexObject for the pattern '^foo$' with IGNORECASE flags represented by an integer-compatible type that preserves the public RegexFlag name across XOR normalization.

Expected output: MigrationWriter.serialize() returns "re.compile('^foo$', re.RegexFlag['IGNORECASE'])" with the import set {'import re'}, and evaluating that expression produces an equivalent regex object. candidate_a instead emits the numeric flag value 2.

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
