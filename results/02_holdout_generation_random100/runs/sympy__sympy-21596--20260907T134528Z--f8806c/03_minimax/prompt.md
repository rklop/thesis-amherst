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
bug in is_subset(Reals)
Solving issue #19513 has given rise to another bug.
Now:
```
In [8]: S1 = imageset(Lambda(n, n + (n - 1)*(n + 1)*I), S.Integers)

In [9]: S1
Out[9]: {n + ⅈ⋅(n - 1)⋅(n + 1) │ n ∊ ℤ}

In [10]: 2 in S1
Out[10]: False

In [11]: 2 in S1.intersect(Reals)
Out[11]: True
```
This output is incorrect.

Correct output is:
```
In [4]: S1
Out[4]: {n + ⅈ⋅(n - 1)⋅(n + 1) │ n ∊ ℤ}

In [5]: 2 in S1
Out[5]: False

In [6]: 2 in S1.intersect(Reals)
Out[6]: False

In [7]: S2 = Reals

In [8]: S1.intersect(S2)
Out[8]: {-1, 1}
```

</issue_statement>

<original_test_patch>
diff --git a/sympy/sets/tests/test_fancysets.py b/sympy/sets/tests/test_fancysets.py
--- a/sympy/sets/tests/test_fancysets.py
+++ b/sympy/sets/tests/test_fancysets.py
@@ -2,8 +2,9 @@
 from sympy.core.expr import unchanged
 from sympy.sets.fancysets import (ImageSet, Range, normalize_theta_set,
                                   ComplexRegion)
-from sympy.sets.sets import (Complement, FiniteSet, Interval, Union, imageset,
+from sympy.sets.sets import (FiniteSet, Interval, Union, imageset,
                              Intersection, ProductSet, Contains)
+from sympy.sets.conditionset import ConditionSet
 from sympy.simplify.simplify import simplify
 from sympy import (S, Symbol, Lambda, symbols, cos, sin, pi, oo, Basic,
                    Rational, sqrt, tan, log, exp, Abs, I, Tuple, eye,
@@ -657,7 +658,23 @@ def test_infinitely_indexed_set_2():
 def test_imageset_intersect_real():
     from sympy import I
     from sympy.abc import n
-    assert imageset(Lambda(n, n + (n - 1)*(n + 1)*I), S.Integers).intersect(S.Reals) == Complement(S.Integers, FiniteSet((-1, 1)))
+    assert imageset(Lambda(n, n + (n - 1)*(n + 1)*I), S.Integers).intersect(S.Reals) == FiniteSet(-1, 1)
+    im = (n - 1)*(n + S.Half)
+    assert imageset(Lambda(n, n + im*I), S.Integers
+        ).intersect(S.Reals) == FiniteSet(1)
+    assert imageset(Lambda(n, n + im*(n + 1)*I), S.Naturals0
+        ).intersect(S.Reals) == FiniteSet(1)
+    assert imageset(Lambda(n, n/2 + im.expand()*I), S.Integers
+        ).intersect(S.Reals) == ImageSet(Lambda(x, x/2), ConditionSet(
+        n, Eq(n**2 - n/2 - S(1)/2, 0), S.Integers))
+    assert imageset(Lambda(n, n/(1/n - 1) + im*(n + 1)*I), S.Integers
+        ).intersect(S.Reals) == FiniteSet(S.Half)
+    assert imageset(Lambda(n, n/(n - 6) +
+        (n - 3)*(n + 1)*I/(2*n + 2)), S.Integers).intersect(
+        S.Reals) == FiniteSet(-1)
+    assert imageset(Lambda(n, n/(n**2 - 9) +
+        (n - 3)*(n + 1)*I/(2*n + 2)), S.Integers).intersect(
+        S.Reals) is S.EmptySet
     s = ImageSet(
         Lambda(n, -I*(I*(2*pi*n - pi/4) + log(Abs(sqrt(-I))))),
         S.Integers)

</original_test_patch>

<candidate_a_patch>
diff --git a/sympy/sets/handlers/intersection.py b/sympy/sets/handlers/intersection.py
index c54dc8f72a..d0af494137 100644
--- a/sympy/sets/handlers/intersection.py
+++ b/sympy/sets/handlers/intersection.py
@@ -1,4 +1,4 @@
-from sympy import (S, Dummy, Lambda, symbols, Interval, Intersection, Set,
+from sympy import (S, Dummy, Lambda, symbols, Interval, Intersection, Set, factor, expand,
                    EmptySet, FiniteSet, Union, ComplexRegion, Mul)
 from sympy.multipledispatch import dispatch
 from sympy.sets.conditionset import ConditionSet
@@ -278,7 +278,9 @@ def intersection_sets(self, other): # noqa:F811
 
     if other == S.Reals:
         from sympy.core.function import expand_complex
+        from sympy import simplify
         from sympy.solvers.solvers import denoms, solve_linear
+        from sympy.solvers.solveset import solveset
         from sympy.core.relational import Eq
         f = self.lamda.expr
         n = self.lamda.variables[0]
@@ -291,7 +293,11 @@ def intersection_sets(self, other): # noqa:F811
 
         re = re.subs(n_, n)
         im = im.subs(n_, n)
+        # Use solveset to find integer solutions for im = 0
+        # This avoids issues with factor() removing factors
         ifree = im.free_symbols
+        # Check if re == n before creating Lambda
+        re_is_n = (re == n)
         lam = Lambda(n, re)
         if im.is_zero:
             # allow re-evaluation
@@ -303,20 +309,70 @@ def intersection_sets(self, other): # noqa:F811
         elif ifree != {n}:
             return None
         else:
-            # univarite imaginary part in same variable
-            x, xis = zip(*[solve_linear(i, 0) for i in Mul.make_args(im) if n in i.free_symbols])
-            if x and all(i == n for i in x):
-                base_set -= FiniteSet(xis)
+            # 
+            # Use numerator to avoid issues with fractions
+            from sympy import fraction
+            num, den = fraction(im)
+            # Solve numerator = 0 to find where im = 0
+            im_solutions = solveset(Eq(num, 0), n, S.Integers)
+            if im_solutions.is_FiniteSet:
+                # Filter to keep only integer solutions
+                integer_solutions = list(im_solutions)
+                if integer_solutions:
+                    # First intersect with base_set
+                    integer_solutions_in_base = base_set.intersection(FiniteSet(*integer_solutions))
+                    if integer_solutions_in_base.is_FiniteSet:
+                        # Check if denominators have n
+                        denominators_with_n = [d for d in denoms(f) if d.has(n)]
+                        
+                        if denominators_with_n:
+                            # Filter out solutions that make denominators zero
+                            valid_solutions = []
+                            for sol in integer_solutions_in_base:
+                                is_valid = True
+                                for denom in denominators_with_n:
+                                    try:
+                                        if denom.subs(n, sol) == 0:
+                                            is_valid = False
+                                            break
+                                    except:
+                                        pass
+                                if is_valid:
+                                    valid_solutions.append(sol)
+                            
+                            if valid_solutions:
+                                # Pass original solutions to imageset, not computed values
+                                return imageset(lam, FiniteSet(*valid_solutions))
+                            else:
+                                return S.EmptySet
+                        else:
+                            # No denominators with n, can return FiniteSet
+                            if re_is_n:
+                                values = [re.subs(n, sol) for sol in integer_solutions_in_base]
+                                base_set = FiniteSet(*values)
+                            else:
+                                # Keep ImageSet representation
+                                base_set = ConditionSet(n, Eq(expand(im), 0), S.Integers)
+                    else:
+                        # Can't handle non-finite sets here
+                        return
+                else:
+                    # No integer solutions, return empty set
+                    return S.EmptySet
             else:
-                base_set -= ConditionSet(n, Eq(im, 0), S.Integers)
+                # Can't solve exactly, use ConditionSet
+                base_set = base_set.intersection(ConditionSet(n, Eq(im, 0), S.Integers))
         # exclude values that make denominators 0
         for i in denoms(f):
             if i.has(n):
-                sol = list(zip(*[solve_linear(i, 0) for i in Mul.make_args(im) if n in i.free_symbols]))
+                sol = list(zip(*[solve_linear(j, 0) for j in Mul.make_args(i) if n in j.free_symbols]))
                 if sol != []:
                     x, xis = sol
-                    if x and all(i == n for i in x):
-                        base_set -= FiniteSet(xis)
+                    if x and all(j == n for j in x):
+                        base_set -= FiniteSet(*xis)
+                    else:
+                        # use solveset for non-linear denominators
+                        base_set -= solveset(Eq(i, 0), n, S.Integers)
                 else:
                     base_set -= ConditionSet(n, Eq(i, 0), S.Integers)
         return imageset(lam, base_set)

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sympy/sets/handlers/intersection.py b/sympy/sets/handlers/intersection.py
--- a/sympy/sets/handlers/intersection.py
+++ b/sympy/sets/handlers/intersection.py
@@ -5,7 +5,7 @@
 from sympy.sets.fancysets import (Integers, Naturals, Reals, Range,
     ImageSet, Rationals)
 from sympy.sets.sets import UniversalSet, imageset, ProductSet
-
+from sympy.simplify.radsimp import numer
 
 @dispatch(ConditionSet, ConditionSet)  # type: ignore # noqa:F811
 def intersection_sets(a, b): # noqa:F811
@@ -280,6 +280,19 @@ def intersection_sets(self, other): # noqa:F811
         from sympy.core.function import expand_complex
         from sympy.solvers.solvers import denoms, solve_linear
         from sympy.core.relational import Eq
+
+        def _solution_union(exprs, sym):
+            # return a union of linear solutions to i in expr;
+            # if i cannot be solved, use a ConditionSet for solution
+            sols = []
+            for i in exprs:
+                x, xis = solve_linear(i, 0, [sym])
+                if x == sym:
+                    sols.append(FiniteSet(xis))
+                else:
+                    sols.append(ConditionSet(sym, Eq(i, 0)))
+            return Union(*sols)
+
         f = self.lamda.expr
         n = self.lamda.variables[0]
 
@@ -303,22 +316,14 @@ def intersection_sets(self, other): # noqa:F811
         elif ifree != {n}:
             return None
         else:
-            # univarite imaginary part in same variable
-            x, xis = zip(*[solve_linear(i, 0) for i in Mul.make_args(im) if n in i.free_symbols])
-            if x and all(i == n for i in x):
-                base_set -= FiniteSet(xis)
-            else:
-                base_set -= ConditionSet(n, Eq(im, 0), S.Integers)
+            # univarite imaginary part in same variable;
+            # use numer instead of as_numer_denom to keep
+            # this as fast as possible while still handling
+            # simple cases
+            base_set &= _solution_union(
+                Mul.make_args(numer(im)), n)
         # exclude values that make denominators 0
-        for i in denoms(f):
-            if i.has(n):
-                sol = list(zip(*[solve_linear(i, 0) for i in Mul.make_args(im) if n in i.free_symbols]))
-                if sol != []:
-                    x, xis = sol
-                    if x and all(i == n for i in x):
-                        base_set -= FiniteSet(xis)
-                else:
-                    base_set -= ConditionSet(n, Eq(i, 0), S.Integers)
+        base_set -= _solution_union(denoms(f), n)
         return imageset(lam, base_set)
 
     elif isinstance(other, Interval):

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_imageset_intersect_real_respects_base_set",
  "specification_gap": "Intersecting an ImageSet with Reals must restrict zero-imaginary-part indices to the ImageSet's original base set. Candidate A replaces the Naturals domain with Integers when the real part is not the identity, introducing values from invalid indices.",
  "input_description": "Create {2*n + I*(n - 1)*(n + 1) | n in Naturals}, intersect it with Reals, and query membership of -2. Producing -2 would require n = -1, which is outside Naturals.",
  "expected_output": "The membership query returns False; intersecting a set with Reals cannot introduce -2 into the original ImageSet.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This checks the general set-intersection invariant through an alternate public ImageSet base domain and a non-identity real mapping. Candidate B preserves the original base-set restriction, while candidate A widens it to Integers.",
  "test_patch": "diff --git a/sympy/sets/tests/test_fancysets.py b/sympy/sets/tests/test_fancysets.py\n--- a/sympy/sets/tests/test_fancysets.py\n+++ b/sympy/sets/tests/test_fancysets.py\n@@ -668,6 +668,13 @@ def test_imageset_intersect_real():\n         Lambda(n, 2*pi*n + pi*Rational(7, 4)), S.Integers)\n \n \n+def test_imageset_intersect_real_respects_base_set():\n+    from sympy.abc import n\n+    values = imageset(\n+        Lambda(n, 2*n + (n - 1)*(n + 1)*I), S.Naturals)\n+    assert -2 not in values.intersect(S.Reals)\n+\n+\n def test_imageset_intersect_interval():\n     from sympy.abc import n\n     f1 = ImageSet(Lambda(n, n*pi), S.Integers)\n",
  "test_command": "python -c \"from sympy.sets.tests.test_fancysets import test_imageset_intersect_real_respects_base_set as test; test()\""
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
/inputs/candidate.patch:41: trailing whitespace.
            # 
/inputs/candidate.patch:56: trailing whitespace.
                        
/inputs/candidate.patch:71: trailing whitespace.
                            
warning: 3 lines add whitespace errors.
[pipeline] checking generated test patch
[pipeline] executing generated test command
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/testbed/sympy/sets/tests/test_fancysets.py", line 675, in test_imageset_intersect_real_respects_base_set
    assert -2 not in values.intersect(S.Reals)
AssertionError
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
[pipeline] test_exit_code=0

```
</validated_execution>
