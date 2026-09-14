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
<candidate_a_patch>
diff --git a/sympy/polys/polytools.py b/sympy/polys/polytools.py
index d5ac051f6..68616816d 100644
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -60,6 +60,7 @@
 
 @public
 class Poly(Expr):
+    _op_priority = 11.0
     """
     Generic class for representing and operating on polynomial expressions.
     Subclasses Expr class.
@@ -391,7 +392,7 @@ def unify(f, g):
     def _unify(f, g):
         g = sympify(g)
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 return f.rep.dom, f.per, f.rep, f.rep.per(f.rep.dom.from_sympy(g))
             except CoercionFailed:
@@ -1333,7 +1334,7 @@ def add(f, g):
         """
         g = sympify(g)
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             return f.add_ground(g)
 
         _, per, F, G = f._unify(g)
@@ -1364,7 +1365,7 @@ def sub(f, g):
         """
         g = sympify(g)
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             return f.sub_ground(g)
 
         _, per, F, G = f._unify(g)
@@ -1395,7 +1396,7 @@ def mul(f, g):
         """
         g = sympify(g)
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             return f.mul_ground(g)
 
         _, per, F, G = f._unify(g)
@@ -3999,7 +4000,7 @@ def __neg__(f):
 
     @_sympifyit('g', NotImplemented)
     def __add__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4009,7 +4010,7 @@ def __add__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __radd__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4019,7 +4020,7 @@ def __radd__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __sub__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4029,7 +4030,7 @@ def __sub__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __rsub__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4039,7 +4040,7 @@ def __rsub__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __mul__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4049,7 +4050,10 @@ def __mul__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __rmul__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
+            # Don't try to convert matrices to Poly - handle element-wise
+            if getattr(g, 'is_Matrix', False):
+                return g._eval_scalar_rmul(f)
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4066,42 +4070,42 @@ def __pow__(f, n):
 
     @_sympifyit('g', NotImplemented)
     def __divmod__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return f.div(g)
 
     @_sympifyit('g', NotImplemented)
     def __rdivmod__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return g.div(f)
 
     @_sympifyit('g', NotImplemented)
     def __mod__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return f.rem(g)
 
     @_sympifyit('g', NotImplemented)
     def __rmod__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return g.rem(f)
 
     @_sympifyit('g', NotImplemented)
     def __floordiv__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return f.quo(g)
 
     @_sympifyit('g', NotImplemented)
     def __rfloordiv__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             g = f.__class__(g, *f.gens)
 
         return g.quo(f)
@@ -4121,7 +4125,7 @@ def __rdiv__(f, g):
     def __eq__(self, other):
         f, g = self, other
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, f.gens, domain=f.get_domain())
             except (PolynomialError, DomainError, CoercionFailed):
@@ -4199,7 +4203,7 @@ def free_symbols(self):
     def __eq__(self, other):
         f, g = self, other
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, f.gens, domain=f.get_domain())
             except (PolynomialError, DomainError, CoercionFailed):
@@ -4225,7 +4229,7 @@ def _strict_eq(f, g):
     def _unify(f, g):
         g = sympify(g)
 
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 return f.rep.dom, f.per, f.rep, f.rep.per(f.rep.dom.from_sympy(g))
             except CoercionFailed:

</candidate_a_patch>
<candidate_b_patch>
diff --git a/sympy/polys/polytools.py b/sympy/polys/polytools.py
index d5ac051f6..3b13ca324 100644
--- a/sympy/polys/polytools.py
+++ b/sympy/polys/polytools.py
@@ -106,6 +106,7 @@ class Poly(Expr):
 
     is_commutative = True
     is_Poly = True
+    _op_priority = 10.001
 
     def __new__(cls, rep, *gens, **args):
         """Create a new polynomial instance out of something useful. """
@@ -4039,7 +4040,7 @@ def __rsub__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __mul__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:
@@ -4049,7 +4050,7 @@ def __mul__(f, g):
 
     @_sympifyit('g', NotImplemented)
     def __rmul__(f, g):
-        if not g.is_Poly:
+        if not getattr(g, "is_Poly", False):
             try:
                 g = f.__class__(g, *f.gens)
             except PolynomialError:

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_poly_equality_with_matrix_operand",
  "specification_gap": "Polynomial equality must safely classify public SymPy operands that do not expose `is_Poly`; an incompatible matrix compares unequal rather than raising `AttributeError`.",
  "input_description": "Evaluate `Poly(x, x) == Matrix([[x]])`, comparing a polynomial with a 1-by-1 mutable matrix.",
  "expected_output": "The comparison returns the Python boolean `False` without raising an exception.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises heterogeneous equality, an alternate public operation affected by the same operand-classification defect. Candidate_a safely handles missing `is_Poly`; candidate_b only hardens multiplication. The command uses SymPy's dependency-free native test runner because pytest is unavailable in the container.",
  "test_patch": "diff --git a/sympy/polys/tests/test_polytools_equality_types.py b/sympy/polys/tests/test_polytools_equality_types.py\nnew file mode 100644\n--- /dev/null\n+++ b/sympy/polys/tests/test_polytools_equality_types.py\n@@ -0,0 +1,6 @@\n+from sympy import Matrix, Poly\n+from sympy.abc import x\n+\n+\n+def test_poly_equality_with_matrix_operand():\n+    assert (Poly(x, x) == Matrix([[x]])) is False\n",
  "test_command": "cd /testbed && python bin/test sympy/polys/tests/test_polytools_equality_types.py --no-colors"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 3,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.689,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.693,
      "log_path": "02_execution/attempt_03/candidate_b.log"
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
cache:              yes
ground types:       python 
numpy:              None
random seed:        51482021
hash randomization: on (PYTHONHASHSEED=4162615039)

sympy/polys/tests/test_polytools_equality_types.py[1] .                     [OK]

================== tests finished: 1 passed, in 0.00 seconds ===================
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
cache:              yes
ground types:       python 
numpy:              None
random seed:        49769536
hash randomization: on (PYTHONHASHSEED=309287933)

sympy/polys/tests/test_polytools_equality_types.py[1] E                   [FAIL]

________________________________________________________________________________
 sympy/polys/tests/test_polytools_equality_types.py:test_poly_equality_with_matrix_operand 
  File "/testbed/sympy/polys/tests/test_polytools_equality_types.py", line 6, in test_poly_equality_with_matrix_operand
    assert (Poly(x, x) == Matrix([[x]])) is False
  File "/testbed/sympy/core/decorators.py", line 91, in __sympifyit_wrapper
    return func(a, b)
  File "/testbed/sympy/polys/polytools.py", line 4125, in __eq__
    if not g.is_Poly:
  File "/testbed/sympy/matrices/matrices.py", line 1855, in __getattr__
    raise AttributeError(
AttributeError: MutableDenseMatrix has no attribute is_Poly.

=========== tests finished: 0 passed, 1 exceptions, in 0.00 seconds ============
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
  "duration_seconds": 1.713,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_poly_equality_with_matrix_operand"
  ],
  "required_any_substrings": [
    "[FAIL]"
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
  "reference_log_sha256": "bf2618b14cf89d3f645381201a70f52cff3a9f49a48f921e745f0e47e2b7236b"
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
cache:              yes
ground types:       python 
numpy:              None
random seed:        26704553
hash randomization: on (PYTHONHASHSEED=877238949)

sympy/polys/tests/test_polytools_equality_types.py[1] E                   [FAIL]

________________________________________________________________________________
 sympy/polys/tests/test_polytools_equality_types.py:test_poly_equality_with_matrix_operand 
  File "/testbed/sympy/polys/tests/test_polytools_equality_types.py", line 6, in test_poly_equality_with_matrix_operand
    assert (Poly(x, x) == Matrix([[x]])) is False
  File "/testbed/sympy/core/decorators.py", line 91, in __sympifyit_wrapper
    return func(a, b)
  File "/testbed/sympy/polys/polytools.py", line 4125, in __eq__
    if not g.is_Poly:
  File "/testbed/sympy/matrices/matrices.py", line 1855, in __getattr__
    raise AttributeError(
AttributeError: MutableDenseMatrix has no attribute is_Poly.

=========== tests finished: 0 passed, 1 exceptions, in 0.00 seconds ============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

</gold_execution_log>
