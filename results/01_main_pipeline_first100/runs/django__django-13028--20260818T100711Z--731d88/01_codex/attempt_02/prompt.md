You are designing one high-signal differentiating input/output
test for SWE-Bench Verified instance django__django-13028.

Work read-only and use only the supplied local files. Do not use web search, GitHub or any other connector, upstream commits, pull requests, historical runs, or gold/reference patches. Treat all external sources as prohibited, even if a tool offers them. Inspect these files:

- `00_inputs/instance.json`: issue statement and original benchmark metadata.
- `00_inputs/candidate_a.patch`: first independently generated solution.
- `00_inputs/candidate_b.patch`: second independently generated solution.
- `workspace/repo`: pristine repository at the instance base commit.

Both candidate patches may pass the existing benchmark tests. Find a semantic
disagreement that reveals a missing part of the intended public specification.
Prefer boundary cases, alternate public entry points, arity/type variants, or
invariants that should hold generally. Avoid implementation-detail assertions,
mock-only checks, timing thresholds, formatting trivia, and tests that merely
encode the gold patch.

Return a minimal test-only unified git diff relative to `workspace/repo`, plus
one targeted shell command that runs that test inside the standard SWE-Bench
container from `/testbed`. The command must return 0 when the test passes and
nonzero when it fails. Do not modify source code. Do not execute either
candidate yourself; the outer pipeline will apply and run the proposal in two
fresh isolated containers.

Explain the concrete input and externally observable expected output. Predict
which candidate should pass, but do not force a split if the evidence is weak.
The final response must conform exactly to the supplied JSON schema.

This is retry 2. The previous proposal did not establish a valid
pass/fail separation. Use this execution feedback and produce a materially
different or corrected test:

<prior_execution_feedback>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.008s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (1 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.007s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (1 silenced).
[pipeline] test_exit_code=0

```
</prior_execution_feedback>

Run artifact root: /home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-13028--20260818T100711Z--731d88
