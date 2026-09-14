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
utils.numberformat.format renders small decimals in exponential notation.
Description
	
When using utils.number_format with decimal_pos, extremely small numbers get displayed using exponential notation.
>>> from django.utils.numberformat import format as nformat
>>> nformat(Decimal('1e-199'), '.', decimal_pos=2)
'0.00'
>>> nformat(Decimal('1e-200'), '.', decimal_pos=2)
'1.00e-200'
This is caused by a hardcoded cut-off point in the internal logic, but I would argue that when a decimal_pos argument is supplied and the number to be formatted is smaller in absolute size than what can be encoded using the provided number of decimal positions, the returned string should be 0.0000...000 instead.

</issue_statement>

<original_test_patch>
diff --git a/tests/utils_tests/test_numberformat.py b/tests/utils_tests/test_numberformat.py
--- a/tests/utils_tests/test_numberformat.py
+++ b/tests/utils_tests/test_numberformat.py
@@ -94,7 +94,7 @@ def test_decimal_numbers(self):
             ('1e-10', 8, '0.00000000'),
             ('1e-11', 8, '0.00000000'),
             ('1' + ('0' * 300), 3, '1.000e+300'),
-            ('0.{}1234'.format('0' * 299), 3, '1.234e-300'),
+            ('0.{}1234'.format('0' * 299), 3, '0.000'),
         ]
         for value, decimal_pos, expected_value in tests:
             with self.subTest(value=value):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/utils/numberformat.py b/django/utils/numberformat.py
index 88b35fc435..f41fddd016 100644
--- a/django/utils/numberformat.py
+++ b/django/utils/numberformat.py
@@ -31,6 +31,16 @@ def format(number, decimal_sep, decimal_pos=None, grouping=0, thousand_sep='',
         # scientific notation to avoid high memory usage in {:f}'.format().
         _, digits, exponent = number.as_tuple()
         if abs(exponent) + len(digits) > 200:
+            # If decimal_pos is specified and the number is smaller than what
+            # can be represented at that precision, format it as zero instead
+            # of using exponential notation.
+            if decimal_pos is not None and number != 0:
+                # The smallest value that can be represented at decimal_pos
+                # decimal places is 10^(-decimal_pos). If the absolute value
+                # of the number is smaller than that, it will round to zero.
+                if abs(number) < Decimal(10) ** (-decimal_pos):
+                    # Return zero with the requested decimal places
+                    return '0' + (decimal_sep + '0' * decimal_pos if decimal_pos else '')
             number = '{:e}'.format(number)
             coefficient, exponent = number.split('e')
             # Format the coefficient.

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/utils/numberformat.py b/django/utils/numberformat.py
--- a/django/utils/numberformat.py
+++ b/django/utils/numberformat.py
@@ -27,6 +27,14 @@ def format(number, decimal_sep, decimal_pos=None, grouping=0, thousand_sep='',
     # sign
     sign = ''
     if isinstance(number, Decimal):
+
+        if decimal_pos is not None:
+            # If the provided number is too small to affect any of the visible
+            # decimal places, consider it equal to '0'.
+            cutoff = Decimal('0.' + '1'.rjust(decimal_pos, '0'))
+            if abs(number) < cutoff:
+                number = Decimal('0')
+
         # Format values with more than 200 digits (an arbitrary cutoff) using
         # scientific notation to avoid high memory usage in {:f}'.format().
         _, digits, exponent = number.as_tuple()

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_negative_decimal_below_precision",
  "specification_gap": "A negative Decimal whose absolute value is smaller than the requested visible precision should be rendered as unsigned zero, regardless of whether it crosses the formatter's internal scientific-notation cutoff.",
  "input_description": "Format Decimal('-9e-19') with decimal separator '.' and decimal_pos=2.",
  "expected_output": "The externally visible string is '0.00', not '-0.00'.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "The issue specifies the comparison in terms of absolute magnitude. Candidate A normalizes to unsigned zero only inside the >200-digit scientific-notation branch, so this input remains '-0.00'. Candidate B applies the rule generally and returns '0.00'. This checks a public sign/precision invariant without asserting the internal cutoff.",
  "test_patch": "diff --git a/tests/utils_tests/test_numberformat.py b/tests/utils_tests/test_numberformat.py\nindex 3d656025ab..d129af992f 100644\n--- a/tests/utils_tests/test_numberformat.py\n+++ b/tests/utils_tests/test_numberformat.py\n@@ -100,6 +100,9 @@ class TestNumberFormat(SimpleTestCase):\n             with self.subTest(value=value):\n                 self.assertEqual(nformat(Decimal(value), '.', decimal_pos), expected_value)\n \n+    def test_negative_decimal_below_precision(self):\n+        self.assertEqual(nformat(Decimal('-9e-19'), '.', decimal_pos=2), '0.00')\n+\n     def test_decimal_subclass(self):\n         class EuroDecimal(Decimal):\n             \"\"\"\n",
  "test_command": "python tests/runtests.py utils_tests.test_numberformat.TestNumberFormat.test_negative_decimal_below_precision"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_negative_decimal_below_precision (utils_tests.test_numberformat.TestNumberFormat)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_numberformat.py", line 104, in test_negative_decimal_below_precision
    self.assertEqual(nformat(Decimal('-9e-19'), '.', decimal_pos=2), '0.00')
AssertionError: '-0.00' != '0.00'
- -0.00
? -
+ 0.00


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
