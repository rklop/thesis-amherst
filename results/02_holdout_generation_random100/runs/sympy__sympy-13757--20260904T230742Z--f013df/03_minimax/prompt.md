Judge the quality of a validated differentiating test for a
SWE-Bench task. Return exactly one JSON object and no Markdown.

Allowed ratings:

- `high_signal`: the test exercises intended public behavior, has a defensible
  oracle, reveals a meaningful missing specification, and is reasonably
  general/minimal rather than tailored to implementation internals.
- `low_signal`: the split is real but primarily reflects brittle internals,
  incidental formatting, undefined behavior, a contrived exploit, unrelated
  regressions, or an oracle not supported by the issue/API contract.
- `ambiguous`: the available evidence does not justify either quality judgment
  without a human deciding an underspecified semantic question.

Do not rate a test high merely because one candidate passes and one fails.
Assess whether the expected output follows from the issue and established API
semantics. Explicitly identify which candidate passed and whether that winner
appears more specification-conformant.

Required JSON keys: `rating`, `confidence`, `summary`, `specification_signal`,
`oracle_quality`, `concerns`, `human_review_questions`. `rating` must be one of
the three allowed values. `confidence` must be a number from 0 to 1. The last
three keys may contain arrays of strings.

<issue_statement>
Multiplying an expression by a Poly does not evaluate when the expression is on the left side of the multiplication
Tested in Python 3.4 64-bit and 3.6 64-bit
Version: 1.1.2.dev0
```
>>> Poly(x)*x
Poly(x**2, x, domain='ZZ')

>>> x*Poly(x)
x*Poly(x, x, domain='ZZ')

>>> -2*Poly(x)
Poly(-2*x, x, domain='ZZ')

>>> S(-2)*Poly(x)
-2*Poly(x, x, domain='ZZ')

>>> Poly(x)*S(-2)
Poly(-2*x, x, domain='ZZ')
```

</issue_statement>

<original_test_patch>
diff --git a/sympy/core/tests/test_match.py b/sympy/core/tests/test_match.py
--- a/sympy/core/tests/test_match.py
+++ b/sympy/core/tests/test_match.py
@@ -134,7 +134,7 @@ def test_mul():
     assert e.match(x**p*exp(x*q)) == {p: 0, q: 1}
 
     e = I*Poly(x, x)
-    assert e.match(I*p) == {p: Poly(x, x)}
+    assert e.match(I*p) == {p: x}
 
 
 def test_mul_noncommutative():
diff --git a/sympy/polys/tests/test_polytools.py b/sympy/polys/tests/test_polytools.py
--- a/sympy/polys/tests/test_polytools.py
+++ b/sympy/polys/tests/test_polytools.py
@@ -677,6 +677,12 @@ def test_Poly_mul():
     assert Poly(x, x) * 2 == Poly(2*x, x)
     assert 2 * Poly(x, x) == Poly(2*x, x)
 
+def test_issue_13079():
+    assert Poly(x)*x == Poly(x**2, x, domain='ZZ')
+    assert x*Poly(x) == Poly(x**2, x, domain='ZZ')
+    assert -2*Poly(x) == Poly(-2*x, x, domain='ZZ')
+    assert S(-2)*Poly(x) == Poly(-2*x, x, domain='ZZ')
+    assert Poly(x)*S(-2) == Poly(-2*x, x, domain='ZZ')
 
 def test_Poly_sqr():
     assert Poly(x*y, x, y).sqr() == Poly(x**2*y**2, x, y)

</original_test_patch>

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
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -106,6 +106,7 @@ class Poly(Expr):
 
     is_commutative = True
     is_Poly = True
+    _op_priority = 10.001
 
     def __new__(cls, rep, *gens, **args):
         """Create a new polynomial instance out of something useful. """

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_poly_is_a_scalar_for_concrete_matrix_multiplication",
  "specification_gap": "Raising Poly's operator priority must fix reflected multiplication by ordinary Expr objects without overriding higher-priority concrete Matrix scalar multiplication. A Poly should remain a scalar when multiplied by a Matrix.",
  "input_description": "Multiply Matrix([[1, x]]) on the right by Poly(x, x).",
  "expected_output": "Matrix([[Poly(x, x), Poly(x**2, x)]]) with no exception. Candidate B's priority 10.001 is below Matrix's 10.01, while candidate A's priority 11 incorrectly routes the entire Matrix through Poly.__rmul__.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises an alternate public operand type and the general operator-priority invariant. It distinguishes a narrowly sufficient priority increase from one that disrupts Matrix multiplication dispatch.",
  "test_patch": "diff --git a/sympy/polys/tests/test_poly_matrix_multiplication.py b/sympy/polys/tests/test_poly_matrix_multiplication.py\nnew file mode 100644\n--- /dev/null\n+++ b/sympy/polys/tests/test_poly_matrix_multiplication.py\n@@ -0,0 +1,7 @@\n+from sympy import Matrix, Poly, Symbol\n+\n+\n+def test_poly_is_a_scalar_for_concrete_matrix_multiplication():\n+    x = Symbol('x')\n+    assert Matrix([[1, x]]) * Poly(x, x) == Matrix(\n+        [[Poly(x, x), Poly(x**2, x)]])\n",
  "test_command": "cd /testbed && python bin/test sympy/polys/tests/test_poly_matrix_multiplication.py"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
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
cache:              yes
ground types:       python 
numpy:              None
random seed:        59295579
hash randomization: on (PYTHONHASHSEED=2948513623)

sympy/polys/tests/test_poly_matrix_multiplication.py[1] E                 [FAIL]

________________________________________________________________________________
 sympy/polys/tests/test_poly_matrix_multiplication.py:test_poly_is_a_scalar_for_concrete_matrix_multiplication 
  File "/testbed/sympy/polys/tests/test_poly_matrix_multiplication.py", line 6, in test_poly_is_a_scalar_for_concrete_matrix_multiplication
    assert Matrix([[1, x]]) * Poly(x, x) == Matrix(
  File "/testbed/sympy/core/decorators.py", line 131, in binary_op_wrapper
    return f(self)
  File "/testbed/sympy/core/decorators.py", line 91, in __sympifyit_wrapper
    return func(a, b)
  File "/testbed/sympy/polys/polytools.py", line 4053, in __rmul__
    if not g.is_Poly:
  File "/testbed/sympy/matrices/matrices.py", line 1855, in __getattr__
    raise AttributeError(
AttributeError: MutableDenseMatrix has no attribute is_Poly.

=========== tests finished: 0 passed, 1 exceptions, in 0.00 seconds ============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
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
cache:              yes
ground types:       python 
numpy:              None
random seed:        84250411
hash randomization: on (PYTHONHASHSEED=1102786415)

sympy/polys/tests/test_poly_matrix_multiplication.py[1] .                   [OK]

================== tests finished: 1 passed, in 0.00 seconds ===================
[pipeline] test_exit_code=0

```
</validated_execution>
