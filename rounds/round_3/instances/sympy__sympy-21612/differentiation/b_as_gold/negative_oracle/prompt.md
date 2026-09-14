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
Latex parsing of fractions yields wrong expression due to missing brackets
Problematic latex expression: `"\\frac{\\frac{a^3+b}{c}}{\\frac{1}{c^2}}"`

is parsed to: `((a**3 + b)/c)/1/(c**2)`.

Expected is: `((a**3 + b)/c)/(1/(c**2))`. 

The missing brackets in the denominator result in a wrong expression.

## Tested on

- 1.8
- 1.6.2

## Reproduce:

```
root@d31ef1c26093:/# python3
Python 3.6.9 (default, Jan 26 2021, 15:33:00)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from sympy.parsing.latex import parse_latex
>>> parse_latex("\\frac{\\frac{a^3+b}{c}}{\\frac{1}{c^2}}")
((a**3 + b)/c)/1/(c**2)



</issue_statement>
<candidate_a_patch>
diff --git a/sympy/printing/str.py b/sympy/printing/str.py
index c3fdcdd435..c7b2c31b88 100644
--- a/sympy/printing/str.py
+++ b/sympy/printing/str.py
@@ -336,6 +336,11 @@ def apow(i):
                             isinstance(item.base, Mul)):
                         # To avoid situations like #14160
                         pow_paren.append(item)
+                    elif isinstance(item.base, Pow) and (item.base.exp.as_coeff_Mul()[0] < 0 or item.base.exp.is_negative):
+                        # Nested reciprocal: e.g., Pow(1/y, -1) -> base is 1/y
+                        # which is Pow(y, -1) and needs parentheses
+                        # Also handles symbolic negative exponents like n (with negative=True)
+                        pow_paren.append(item)
                     b.append(item.base)
             elif item.is_Rational and item is not S.Infinity:
                 if item.p != 1:

</candidate_a_patch>
<candidate_b_patch>
diff --git a/sympy/printing/str.py b/sympy/printing/str.py
index c3fdcdd435..e71baf5d75 100644
--- a/sympy/printing/str.py
+++ b/sympy/printing/str.py
@@ -332,9 +332,12 @@ def apow(i):
                 if item.exp is not S.NegativeOne:
                     b.append(apow(item))
                 else:
-                    if (len(item.args[0].args) != 1 and
-                            isinstance(item.base, Mul)):
-                        # To avoid situations like #14160
+                    if ((len(item.args[0].args) != 1 and
+                            isinstance(item.base, Mul)) or
+                            # To avoid situations like #21537
+                            (isinstance(item.base, Pow) and
+                                (bool(item.base.exp.as_coeff_Mul()[0] < 0) or
+                                    item.base.exp.is_negative is True))):
                         pow_paren.append(item)
                     b.append(item.base)
             elif item.is_Rational and item is not S.Infinity:

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_nested_reciprocal_with_conditional_exponent_sign",
  "specification_gap": "String printing a nested reciprocal should apply special parentheses only when the inner exponent is definitely negative; an unresolved symbolic sign condition must not be coerced to a Python boolean.",
  "input_description": "Construct the unevaluated quotient x/(y**e), where e is a custom symbolic exponent whose `is_negative` result is the unresolved relation `x < 0`, and pass it to the public `str()` printer.",
  "expected_output": "Printing completes without an exception and returns exactly `x/y**ConditionalSignExponent()`.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate A truth-tests the unresolved relation and raises TypeError. Candidate B requires negativity to be literal `True`, so it safely prints the expression. The new-file diff was also checked successfully with `git apply --check`.",
  "test_patch": "diff --git a/sympy/printing/tests/test_str_nested_reciprocal.py b/sympy/printing/tests/test_str_nested_reciprocal.py\nnew file mode 100644\n--- /dev/null\n+++ b/sympy/printing/tests/test_str_nested_reciprocal.py\n@@ -0,0 +1,19 @@\n+from sympy import Mul, Pow, Symbol\n+from sympy.core import Expr\n+\n+\n+def test_nested_reciprocal_with_conditional_exponent_sign():\n+    x = Symbol('x')\n+    y = Symbol('y')\n+\n+    class ConditionalSignExponent(Expr):\n+        is_commutative = True\n+\n+        @property\n+        def is_negative(self):\n+            return x < 0\n+\n+    exponent = ConditionalSignExponent()\n+    denominator = Pow(y, exponent, evaluate=False)\n+    quotient = Mul(x, Pow(denominator, -1, evaluate=False), evaluate=False)\n+    assert str(quotient) == 'x/y**ConditionalSignExponent()'\n",
  "test_command": "cd /testbed && python bin/test sympy/printing/tests/test_str_nested_reciprocal.py --no-colors"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 3,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.947,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.906,
      "log_path": "02_execution/attempt_03/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              yes
ground types:       python 
numpy:              None
random seed:        35413044
hash randomization: on (PYTHONHASHSEED=2896421331)

sympy/printing/tests/test_str_nested_reciprocal.py[1] E                   [FAIL]

________________________________________________________________________________
 sympy/printing/tests/test_str_nested_reciprocal.py:test_nested_reciprocal_with_conditional_exponent_sign 
Traceback (most recent call last):
  File "/testbed/sympy/printing/tests/test_str_nested_reciprocal.py", line 19, in test_nested_reciprocal_with_conditional_exponent_sign
    assert str(quotient) == 'x/y**ConditionalSignExponent()'
  File "/testbed/sympy/core/_print_helpers.py", line 29, in __str__
    return sstr(self, order=None)
  File "/testbed/sympy/printing/printer.py", line 373, in __call__
    return self.__wrapped__(*args, **kwargs)
  File "/testbed/sympy/printing/str.py", line 973, in sstr
    s = p.doprint(expr)
  File "/testbed/sympy/printing/printer.py", line 291, in doprint
    return self._str(self._print(expr))
  File "/testbed/sympy/printing/printer.py", line 329, in _print
    return getattr(self, printmethod)(expr, **kwargs)
  File "/testbed/sympy/printing/str.py", line 339, in _print_Mul
    elif isinstance(item.base, Pow) and (item.base.exp.as_coeff_Mul()[0] < 0 or item.base.exp.is_negative):
  File "/testbed/sympy/core/relational.py", line 398, in __bool__
    raise TypeError("cannot determine truth value of Relational")
TypeError: cannot determine truth value of Relational

=========== tests finished: 0 passed, 1 exceptions, in 0.00 seconds ============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              yes
ground types:       python 
numpy:              None
random seed:        42232528
hash randomization: on (PYTHONHASHSEED=1921895235)

sympy/printing/tests/test_str_nested_reciprocal.py[1] .                     [OK]

================== tests finished: 1 passed, in 0.00 seconds ===================
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/sympy/printing/str.py b/sympy/printing/str.py
--- a/sympy/printing/str.py
+++ b/sympy/printing/str.py
@@ -333,7 +333,7 @@ def apow(i):
                     b.append(apow(item))
                 else:
                     if (len(item.args[0].args) != 1 and
-                            isinstance(item.base, Mul)):
+                            isinstance(item.base, (Mul, Pow))):
                         # To avoid situations like #14160
                         pow_paren.append(item)
                     b.append(item.base)

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.91,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_nested_reciprocal_with_conditional_exponent_sign"
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
  "reference_log_sha256": "a72908d82a0dfc1f6b2a988f64df73bba66660ef82861edad1ab4d070ba8773f"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
============================= test process starts ==============================
executable:         /opt/miniconda3/envs/testbed/bin/python  (3.9.20-final-0) [CPython]
architecture:       64-bit
cache:              yes
ground types:       python 
numpy:              None
random seed:        78014872
hash randomization: on (PYTHONHASHSEED=2686865453)

sympy/printing/tests/test_str_nested_reciprocal.py[1] F                   [FAIL]

________________________________________________________________________________
 sympy/printing/tests/test_str_nested_reciprocal.py:test_nested_reciprocal_with_conditional_exponent_sign 
Traceback (most recent call last):
  File "/testbed/sympy/printing/tests/test_str_nested_reciprocal.py", line 19, in test_nested_reciprocal_with_conditional_exponent_sign
    assert str(quotient) == 'x/y**ConditionalSignExponent()'
AssertionError

============= tests finished: 0 passed, 1 failed, in 0.00 seconds ==============
DO *NOT* COMMIT!
[pipeline] test_exit_code=1

</gold_execution_log>
