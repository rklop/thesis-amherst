You are designing one high-signal differentiating input/output
test for SWE-Bench Verified instance sphinx-doc__sphinx-11445.

Work read-only and use only the supplied local files. Do not use web search, GitHub or any other connector, upstream commits, pull requests, historical runs, or any gold/reference patch beyond the one explicitly supplied in `00_inputs`. Treat all other external sources as prohibited, even if a tool offers them. Inspect
these files:

- `00_inputs/instance.json`: issue statement and original benchmark metadata.
- `00_inputs/designated_gold.patch`: the supplied gold patch (internally `candidate_b`).
- `00_inputs/generated_candidate.patch`: the generated candidate patch (internally `candidate_a`).
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

- the supplied gold patch (`candidate_b`) passes; and
- the generated candidate patch (`candidate_a`) fails.

The outer pipeline will reject an opposite-direction split even if it is a
real separation. Set `expected_passing_candidate` to
`candidate_b`. The expected output and assertions must follow
the supplied gold behavior and the issue's public contract. Do not weaken,
reverse, or make the oracle vacuous merely to force a split.


This is retry 2. The previous proposal did not establish a valid
pass/fail separation. Use this execution feedback and produce a materially
different or corrected test:

<prior_execution_feedback>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
______________ test_RSTParser_prolog_precedes_indented_field_list ______________

app = <SphinxTestApp buildername='html'>

    @pytest.mark.sphinx(testroot='basic')
    def test_RSTParser_prolog_precedes_indented_field_list(app):
        app.env.config.rst_prolog = 'Prolog paragraph.'
        document = new_document('dummy.rst')
        parser = RSTParser()
        parser.set_application(app)
    
>       parser.parse('   :note: nested field\n', document)

tests/test_parser.py:67: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <sphinx.parsers.RSTParser object at 0x72d3051d0dc0>
inputstring = '   :note: nested field\n', document = <document: >

    def parse(self, inputstring: str | StringList, document: nodes.document) -> None:
        """Parse text and generate a document tree."""
        self.setup_parse(inputstring, document)  # type: ignore
        self.statemachine = states.RSTStateMachine(
            state_classes=self.state_classes,
            initial_state=self.initial_state,
            debug=document.reporter.debug_flag)
    
        # preprocess inputstring
        if isinstance(inputstring, str):
            lines = docutils.statemachine.string2lines(
>               inputstring, tab_width=document.settings.tab_width,
                convert_whitespace=True)
E           AttributeError: 'Values' object has no attribute 'tab_width'

sphinx/parsers.py:72: AttributeError
--------------------------- Captured stdout teardown ---------------------------
# testroot: root
# builder: html
# srcdir: /tmp/pytest-of-root/pytest-0/basic
# outdir: /tmp/pytest-of-root/pytest-0/basic/_build/html
# status: 
[01mRunning Sphinx v7.1.0+/57b0661d9[39;49;00m

# warning: 

=========================== short test summary info ============================
FAILED tests/test_parser.py::test_RSTParser_prolog_precedes_indented_field_list
1 failed in 0.37s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
______________ test_RSTParser_prolog_precedes_indented_field_list ______________

app = <SphinxTestApp buildername='html'>

    @pytest.mark.sphinx(testroot='basic')
    def test_RSTParser_prolog_precedes_indented_field_list(app):
        app.env.config.rst_prolog = 'Prolog paragraph.'
        document = new_document('dummy.rst')
        parser = RSTParser()
        parser.set_application(app)
    
>       parser.parse('   :note: nested field\n', document)

tests/test_parser.py:67: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <sphinx.parsers.RSTParser object at 0x7cdaff3910d0>
inputstring = '   :note: nested field\n', document = <document: >

    def parse(self, inputstring: str | StringList, document: nodes.document) -> None:
        """Parse text and generate a document tree."""
        self.setup_parse(inputstring, document)  # type: ignore
        self.statemachine = states.RSTStateMachine(
            state_classes=self.state_classes,
            initial_state=self.initial_state,
            debug=document.reporter.debug_flag)
    
        # preprocess inputstring
        if isinstance(inputstring, str):
            lines = docutils.statemachine.string2lines(
>               inputstring, tab_width=document.settings.tab_width,
                convert_whitespace=True)
E           AttributeError: 'Values' object has no attribute 'tab_width'

sphinx/parsers.py:72: AttributeError
--------------------------- Captured stdout teardown ---------------------------
# testroot: root
# builder: html
# srcdir: /tmp/pytest-of-root/pytest-0/basic
# outdir: /tmp/pytest-of-root/pytest-0/basic/_build/html
# status: 
[01mRunning Sphinx v7.1.0+/57b0661d9[39;49;00m

# warning: 

=========================== short test summary info ============================
FAILED tests/test_parser.py::test_RSTParser_prolog_precedes_indented_field_list
1 failed in 0.35s
[pipeline] test_exit_code=1

```
</prior_execution_feedback>

Run artifact root: /home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-11445/b_as_gold/differentiation/sphinx-doc__sphinx-11445--20260908T133944Z--a489c2
