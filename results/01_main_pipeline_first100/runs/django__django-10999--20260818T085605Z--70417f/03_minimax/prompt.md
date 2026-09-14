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
Fix parse_duration() for some negative durations
Description
	
The ​https://docs.djangoproject.com/en/2.1/_modules/django/utils/dateparse/ defines:
standard_duration_re = re.compile(
	r'^'
	r'(?:(?P<days>-?\d+) (days?, )?)?'
	r'((?:(?P<hours>-?\d+):)(?=\d+:\d+))?'
	r'(?:(?P<minutes>-?\d+):)?'
	r'(?P<seconds>-?\d+)'
	r'(?:\.(?P<microseconds>\d{1,6})\d{0,6})?'
	r'$'
)
that doesn't match to negative durations, because of the <hours> definition final (lookahead) part does not have '-?' in it. The following will work:
	r'((?:(?P<hours>-?\d+):)(?=-?\d+:-?\d+))?'
(Thanks to Konstantin Senichev for finding the fix.)

</issue_statement>

<original_test_patch>
diff --git a/tests/utils_tests/test_dateparse.py b/tests/utils_tests/test_dateparse.py
--- a/tests/utils_tests/test_dateparse.py
+++ b/tests/utils_tests/test_dateparse.py
@@ -113,9 +113,12 @@ def test_negative(self):
         test_values = (
             ('-4 15:30', timedelta(days=-4, minutes=15, seconds=30)),
             ('-172800', timedelta(days=-2)),
-            ('-15:30', timedelta(minutes=-15, seconds=30)),
-            ('-1:15:30', timedelta(hours=-1, minutes=15, seconds=30)),
+            ('-15:30', timedelta(minutes=-15, seconds=-30)),
+            ('-1:15:30', timedelta(hours=-1, minutes=-15, seconds=-30)),
             ('-30.1', timedelta(seconds=-30, milliseconds=-100)),
+            ('-00:01:01', timedelta(minutes=-1, seconds=-1)),
+            ('-01:01', timedelta(seconds=-61)),
+            ('-01:-01', None),
         )
         for source, expected in test_values:
             with self.subTest(source=source):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/utils/dateparse.py b/django/utils/dateparse.py
index 8d08b7d1d3..da2bf0f276 100644
--- a/django/utils/dateparse.py
+++ b/django/utils/dateparse.py
@@ -29,7 +29,7 @@ datetime_re = re.compile(
 standard_duration_re = re.compile(
     r'^'
     r'(?:(?P<days>-?\d+) (days?, )?)?'
-    r'((?:(?P<hours>-?\d+):)(?=\d+:\d+))?'
+    r'((?:(?P<hours>-?\d+):)(?=-?\d+:-?\d+))?'
     r'(?:(?P<minutes>-?\d+):)?'
     r'(?P<seconds>-?\d+)'
     r'(?:\.(?P<microseconds>\d{1,6})\d{0,6})?'
@@ -138,6 +138,27 @@ def parse_duration(value):
         kw = match.groupdict()
         days = datetime.timedelta(float(kw.pop('days', 0) or 0))
         sign = -1 if kw.pop('sign', '+') == '-' else 1
+        # For standard_duration_re, propagate sign from hours/minutes to seconds/minutes
+        # when there's a leading negative (e.g., '-15:30' means -15 minutes and -30 seconds)
+        # Also check for invalid format like '-01:-01' where both minutes and seconds have signs
+        if 'sign' not in kw and kw.get('seconds'):
+            has_leading_neg = (kw.get('hours') and kw['hours'].startswith('-')) or \
+                             (kw.get('minutes') and kw['minutes'].startswith('-'))
+            seconds_has_sign = kw['seconds'].startswith('-') or kw['seconds'].startswith('+')
+            minutes_has_sign = kw.get('minutes') and (kw['minutes'].startswith('-') or kw['minutes'].startswith('+'))
+            # Invalid format: both minutes and seconds have signs (like '-01:-01')
+            if minutes_has_sign and seconds_has_sign:
+                return None
+            if has_leading_neg:
+                # Invalid if seconds already have a different sign
+                if seconds_has_sign and not kw['seconds'].startswith('-'):
+                    return None
+                if not seconds_has_sign:
+                    kw['seconds'] = '-' + kw['seconds']
+                # Also propagate to minutes if hours has sign
+                if kw.get('hours') and kw['hours'].startswith('-') and kw.get('minutes'):
+                    if not kw['minutes'].startswith('-'):
+                        kw['minutes'] = '-' + kw['minutes']
         if kw.get('microseconds'):
             kw['microseconds'] = kw['microseconds'].ljust(6, '0')
         if kw.get('seconds') and kw.get('microseconds') and kw['seconds'].startswith('-'):

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/utils/dateparse.py b/django/utils/dateparse.py
--- a/django/utils/dateparse.py
+++ b/django/utils/dateparse.py
@@ -29,9 +29,10 @@
 standard_duration_re = re.compile(
     r'^'
     r'(?:(?P<days>-?\d+) (days?, )?)?'
-    r'((?:(?P<hours>-?\d+):)(?=\d+:\d+))?'
-    r'(?:(?P<minutes>-?\d+):)?'
-    r'(?P<seconds>-?\d+)'
+    r'(?P<sign>-?)'
+    r'((?:(?P<hours>\d+):)(?=\d+:\d+))?'
+    r'(?:(?P<minutes>\d+):)?'
+    r'(?P<seconds>\d+)'
     r'(?:\.(?P<microseconds>\d{1,6})\d{0,6})?'
     r'$'
 )

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_negative_sign_after_colon",
  "specification_gap": "Only a leading minus sign may negate a standard duration. Candidate A also accepts a minus sign on an individual component after a colon.",
  "input_description": "Call parse_duration('00:01:-01'), where the seconds component has an internal sign.",
  "expected_output": "parse_duration() returns None because the input is not a valid standard, ISO 8601, or PostgreSQL duration. Candidate A instead returns timedelta(seconds=59).",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This boundary case checks the general sign-position invariant. It exposes Candidate A's acceptance of individually signed components, beyond the whole-duration negative examples covered by the benchmark.",
  "test_patch": "diff --git a/tests/utils_tests/test_dateparse.py b/tests/utils_tests/test_dateparse.py\n--- a/tests/utils_tests/test_dateparse.py\n+++ b/tests/utils_tests/test_dateparse.py\n@@ -121,6 +121,9 @@ class DurationParseTests(unittest.TestCase):\n             with self.subTest(source=source):\n                 self.assertEqual(parse_duration(source), expected)\n \n+    def test_negative_sign_after_colon(self):\n+        self.assertIsNone(parse_duration('00:01:-01'))\n+\n     def test_iso_8601(self):\n         test_values = (\n             ('P4Y', None),\n",
  "test_command": "./tests/runtests.py utils_tests.test_dateparse.DurationParseTests.test_negative_sign_after_colon"
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
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_negative_sign_after_colon (utils_tests.test_dateparse.DurationParseTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_dateparse.py", line 125, in test_negative_sign_after_colon
    self.assertIsNone(parse_duration('00:01:-01'))
AssertionError: datetime.timedelta(0, 59) is not None

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
