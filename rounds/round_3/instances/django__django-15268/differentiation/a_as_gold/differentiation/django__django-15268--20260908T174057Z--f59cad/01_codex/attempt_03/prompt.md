You are designing one high-signal differentiating input/output
test for SWE-Bench Verified instance django__django-15268.

Work read-only and use only the supplied local files. Do not use web search, GitHub or any other connector, upstream commits, pull requests, historical runs, or any gold/reference patch beyond the one explicitly supplied in `00_inputs`. Treat all other external sources as prohibited, even if a tool offers them. Inspect
these files:

- `00_inputs/instance.json`: issue statement and original benchmark metadata.
- `00_inputs/designated_gold.patch`: the supplied gold patch (internally `candidate_a`).
- `00_inputs/generated_candidate.patch`: the generated candidate patch (internally `candidate_b`).
- `workspace/repo`: pristine repository at the instance base commit.

The supplied gold patch is the authoritative reference solution for this
task. The other patch is a generated candidate that already passes the existing
benchmark tests. Do not independently relabel, rank, or choose between the
patches.

Find a semantic disagreement that exposes a missing defect in the generated
candidate relative to the supplied gold patch. Use that disagreement to extract
one useful, evidence-grounded piece of semantic information that clarifies the
issue specification, and express it through the generated test. Keep this
clarification narrow; do not attempt to write a comprehensive specification.
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

This run has a strict gold-versus-candidate acceptance rule. Produce only a
test with a defensible expected behavior for which:

- the supplied gold patch (`candidate_a`) passes; and
- the generated candidate patch (`candidate_b`) fails.

The outer pipeline will reject an opposite-direction split even if it is a
real separation. Set `expected_passing_candidate` to
`candidate_a`. The expected output and assertions must follow
the supplied gold behavior and the issue's public contract. Do not weaken,
reverse, or make the oracle vacuous merely to force a split.


This is retry 3. The previous proposal did not establish a valid
pass/fail separation. Use this execution feedback and produce a materially
different or corrected test:

<prior_execution_feedback>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_alter_together_subclass_preserves_state (migrations.test_optimizer.OptimizerTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_optimizer.py", line 241, in test_alter_together_subclass_preserves_state
    self.assertEqual(
AssertionError: Items in the first set but not the second:
('initial',)
Items in the second set but not the first:
('intermediate',)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_alter_together_subclass_preserves_state (migrations.test_optimizer.OptimizerTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_optimizer.py", line 241, in test_alter_together_subclass_preserves_state
    self.assertEqual(
AssertionError: Items in the first set but not the second:
('initial',)
Items in the second set but not the first:
('intermediate',)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```
</prior_execution_feedback>

Run artifact root: /home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15268/a_as_gold/differentiation/django__django-15268--20260908T174057Z--f59cad
