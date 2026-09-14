Act as a careful human software engineer reviewing a potential
negative specification test. Return exactly one JSON object and no Markdown.

The generated test separates two plausible patches, but the actual SWE-bench
gold patch executes and fails it. The question is not whether the assertion
should pass. The question is whether this failure reliably specifies behavior
that a final patch must NOT be required to implement. Extract only a narrow,
implementation-independent negative constraint supported by the issue and gold
evidence. Do not infer that every gold failure is useful.

Use this policy:
- `accept` only when the target test demonstrably ran, its failure is behavioral
  (not collection/setup/infrastructure), and the assertion captures a plausible,
  issue-relevant behavior that the gold evidence specifically declines to require.
- `reject` for infrastructure or collection failures, incidental output, private
  implementation coupling, unrelated behavior, unstable/undefined behavior, or
  when failure does not justify a reusable negative constraint.
- `human_review` only for a consequential ambiguity not resolvable from evidence.

eligible_failure_fingerprint=true. If false, do not accept.
An arbitrary nonzero exit is never sufficient.

Required JSON keys: `decision`, `confidence`, `negative_specification`,
`forbidden_behavior`, `gold_failure_interpretation`, `specification_value`,
`decision_rationale`, `concerns`, `human_review_questions`.

<issue_statement>

</issue_statement>
<candidate_a_patch>
diff --git a/sympy/polys/polytools.py b/sympy/polys/polytools.py
index d5ac051f6..66fe8329b 100644
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -106,6 +106,7 @@ class Poly(Expr):
 
     is_commutative = True
     is_Poly = True
+    _op_priority = 11.0
 
     def __new__(cls, rep, *gens, **args):
         """Create a new polynomial instance out of something useful. """

</candidate_a_patch>
<candidate_b_patch>
diff --git a/sympy/polys/polytools.py b/sympy/polys/polytools.py
index d5ac051f6..30e0685ca 100644
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -60,6 +60,7 @@
 
 @public
 class Poly(Expr):
+    _op_priority = 10.001
     """
     Generic class for representing and operating on polynomial expressions.
     Subclasses Expr class.

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_Poly_mul_immutable_matrix",
  "specification_gap": "Left-hand multiplication must also dispatch to Poly when the expression is an ImmutableMatrix, an Expr-derived public type with its own arithmetic precedence.",
  "input_description": "Multiply the one-element expression ImmutableMatrix([[2]]) on the left by Poly(x, x).",
  "expected_output": "The result is Poly(2*x, x, domain='ZZ'), rather than an ImmutableMatrix containing that polynomial. Candidate_b only ties ImmutableMatrix's precedence and therefore leaves the matrix as the outer result.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises the exact precedence boundary omitted by candidate_b while asserting only the public result's value and type. Ordinary Symbol and Integer operands are below both proposed Poly precedences and cannot expose the defect.",
  "test_patch": "diff --git a/sympy/polys/tests/test_polytools.py b/sympy/polys/tests/test_polytools.py\n--- a/sympy/polys/tests/test_polytools.py\n+++ b/sympy/polys/tests/test_polytools.py\n@@ -675,7 +675,13 @@ def test_Poly_mul():\n     assert Poly(1, x) * sin(x) == sin(x)\n \n     assert Poly(x, x) * 2 == Poly(2*x, x)\n     assert 2 * Poly(x, x) == Poly(2*x, x)\n \n \n+def test_Poly_mul_immutable_matrix():\n+    from sympy import ImmutableMatrix\n+\n+    assert ImmutableMatrix([[2]]) * Poly(x, x) == Poly(2*x, x)\n+\n+\n def test_Poly_sqr():\n     assert Poly(x*y, x, y).sqr() == Poly(x**2*y**2, x, y)\n",
  "test_command": "cd /testbed && python bin/test -C --no-colors -k test_Poly_mul_immutable_matrix sympy/polys/tests/test_polytools.py"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.942,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.894,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/solvers/diophantine.py:3188: SyntaxWarning: "is" with a literal. Did you mean "=="?
  if feasible is 1:  # it's prime and k == 2
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              no
ground types:       python 
numpy:              None
random seed:        27876019
hash randomization: on (PYTHONHASHSEED=3521171896)

sympy/polys/tests/test_polytools.py[1] .                                    [OK]

================== tests finished: 1 passed, in 0.17 seconds ===================
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/solvers/diophantine.py:3188: SyntaxWarning: "is" with a literal. Did you mean "=="?
  if feasible is 1:  # it's prime and k == 2
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              no
ground types:       python 
numpy:              None
random seed:        48016106
hash randomization: on (PYTHONHASHSEED=920268142)

sympy/polys/tests/test_polytools.py[1] F                                  [FAIL]

________________________________________________________________________________
______ sympy/polys/tests/test_polytools.py:test_Poly_mul_immutable_matrix ______
  File "/testbed/sympy/polys/tests/test_polytools.py", line 684, in test_Poly_mul_immutable_matrix
    assert ImmutableMatrix([[2]]) * Poly(x, x) == Poly(2*x, x)
AssertionError

============= tests finished: 0 passed, 1 failed, in 0.17 seconds ==============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/sympy/polys/polytools.py b/sympy/polys/polytools.py
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -106,6 +106,7 @@ class Poly(Expr):
 
     is_commutative = True
     is_Poly = True
+    _op_priority = 10.001
 
     def __new__(cls, rep, *gens, **args):
         """Create a new polynomial instance out of something useful. """

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.941,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_Poly_mul_immutable_matrix"
  ],
  "required_any_substrings": [
    "AssertionError",
    "[FAIL]",
    " failed,"
  ],
  "forbidden_substrings": [
    "collected 0 items",
    "no tests ran",
    "ERROR collecting",
    "command not found",
    "No such file or directory",
    "Could not find a version that satisfies the requirement",
    "Temporary failure in name resolution"
  ],
  "reference_returncode": 1,
  "reference_log_sha256": "364ca8faac728f5157520090b3fd52e728ef50af88a693b4b8adc51cbe719b93"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/solvers/diophantine.py:3188: SyntaxWarning: "is" with a literal. Did you mean "=="?
  if feasible is 1:  # it's prime and k == 2
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
/testbed/sympy/core/basic.py:3: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Mapping, defaultdict
/testbed/sympy/core/containers.py:271: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  class OrderedSet(collections.MutableSet):
/testbed/sympy/plotting/plot.py:28: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated since Python 3.3, and in 3.10 it will stop working
  from collections import Callable
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              no
ground types:       python 
numpy:              None
random seed:        64577826
hash randomization: on (PYTHONHASHSEED=2822290274)

sympy/polys/tests/test_polytools.py[1] F                                  [FAIL]

________________________________________________________________________________
______ sympy/polys/tests/test_polytools.py:test_Poly_mul_immutable_matrix ______
  File "/testbed/sympy/polys/tests/test_polytools.py", line 684, in test_Poly_mul_immutable_matrix
    assert ImmutableMatrix([[2]]) * Poly(x, x) == Poly(2*x, x)
AssertionError

============= tests finished: 0 passed, 1 failed, in 0.17 seconds ==============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

</gold_execution_log>
