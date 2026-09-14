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
django.utils.http.parse_http_date two digit year check is incorrect
Description
	 
		(last modified by Ad Timmering)
	 
RFC 850 does not mention this, but in RFC 7231 (and there's something similar in RFC 2822), there's the following quote:
Recipients of a timestamp value in rfc850-date format, which uses a
two-digit year, MUST interpret a timestamp that appears to be more
than 50 years in the future as representing the most recent year in
the past that had the same last two digits.
Current logic is hard coded to consider 0-69 to be in 2000-2069, and 70-99 to be 1970-1999, instead of comparing versus the current year.

</issue_statement>

<original_test_patch>
diff --git a/tests/utils_tests/test_http.py b/tests/utils_tests/test_http.py
--- a/tests/utils_tests/test_http.py
+++ b/tests/utils_tests/test_http.py
@@ -1,5 +1,6 @@
 import unittest
 from datetime import datetime
+from unittest import mock
 
 from django.test import SimpleTestCase, ignore_warnings
 from django.utils.datastructures import MultiValueDict
@@ -316,9 +317,27 @@ def test_parsing_rfc1123(self):
         parsed = parse_http_date('Sun, 06 Nov 1994 08:49:37 GMT')
         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))
 
-    def test_parsing_rfc850(self):
-        parsed = parse_http_date('Sunday, 06-Nov-94 08:49:37 GMT')
-        self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))
+    @mock.patch('django.utils.http.datetime.datetime')
+    def test_parsing_rfc850(self, mocked_datetime):
+        mocked_datetime.side_effect = datetime
+        mocked_datetime.utcnow = mock.Mock()
+        utcnow_1 = datetime(2019, 11, 6, 8, 49, 37)
+        utcnow_2 = datetime(2020, 11, 6, 8, 49, 37)
+        utcnow_3 = datetime(2048, 11, 6, 8, 49, 37)
+        tests = (
+            (utcnow_1, 'Tuesday, 31-Dec-69 08:49:37 GMT', datetime(2069, 12, 31, 8, 49, 37)),
+            (utcnow_1, 'Tuesday, 10-Nov-70 08:49:37 GMT', datetime(1970, 11, 10, 8, 49, 37)),
+            (utcnow_1, 'Sunday, 06-Nov-94 08:49:37 GMT', datetime(1994, 11, 6, 8, 49, 37)),
+            (utcnow_2, 'Wednesday, 31-Dec-70 08:49:37 GMT', datetime(2070, 12, 31, 8, 49, 37)),
+            (utcnow_2, 'Friday, 31-Dec-71 08:49:37 GMT', datetime(1971, 12, 31, 8, 49, 37)),
+            (utcnow_3, 'Sunday, 31-Dec-00 08:49:37 GMT', datetime(2000, 12, 31, 8, 49, 37)),
+            (utcnow_3, 'Friday, 31-Dec-99 08:49:37 GMT', datetime(1999, 12, 31, 8, 49, 37)),
+        )
+        for utcnow, rfc850str, expected_date in tests:
+            with self.subTest(rfc850str=rfc850str):
+                mocked_datetime.utcnow.return_value = utcnow
+                parsed = parse_http_date(rfc850str)
+                self.assertEqual(datetime.utcfromtimestamp(parsed), expected_date)
 
     def test_parsing_asctime(self):
         parsed = parse_http_date('Sun Nov  6 08:49:37 1994')

</original_test_patch>

<candidate_a_patch>
diff --git a/django/utils/http.py b/django/utils/http.py
index 572cfb4347..1f0176808f 100644
--- a/django/utils/http.py
+++ b/django/utils/http.py
@@ -176,10 +176,10 @@ def parse_http_date(date):
     try:
         year = int(m.group('year'))
         if year < 100:
-            if year < 70:
-                year += 2000
-            else:
-                year += 1900
+            year += 2000
+            current_year = datetime.datetime.utcnow().year
+            if year - current_year > 50:
+                year -= 100
         month = MONTHS.index(m.group('mon').lower()) + 1
         day = int(m.group('day'))
         hour = int(m.group('hour'))

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/utils/http.py b/django/utils/http.py
--- a/django/utils/http.py
+++ b/django/utils/http.py
@@ -176,10 +176,14 @@ def parse_http_date(date):
     try:
         year = int(m.group('year'))
         if year < 100:
-            if year < 70:
-                year += 2000
+            current_year = datetime.datetime.utcnow().year
+            current_century = current_year - (current_year % 100)
+            if year - (current_year % 100) > 50:
+                # year that appears to be more than 50 years in the future are
+                # interpreted as representing the past.
+                year += current_century - 100
             else:
-                year += 1900
+                year += current_century
         month = MONTHS.index(m.group('mon').lower()) + 1
         day = int(m.group('day'))
         hour = int(m.group('hour'))

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_parsing_rfc850_at_next_century_boundary",
  "specification_gap": "Two-digit RFC 850 years must be interpreted relative to the current century, including after a century rollover. A year exactly 50 years in the future must remain in the future because only dates more than 50 years ahead are rolled back.",
  "input_description": "Freeze the current UTC time at 2100-11-06 08:49:37 and parse the RFC 850 date \"Sunday, 06-Nov-50 08:49:37 GMT\".",
  "expected_output": "parse_http_date() returns an epoch timestamp that converts to 2150-11-06 08:49:37 UTC. Candidate A instead resolves the year against a hard-coded 2000 base and produces 2050.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This combines a century rollover with the exact 50-year boundary. It exercises externally visible date parsing while controlling the clock, and exposes whether the implementation is genuinely relative to the current year or only happens to work during the 2000s.",
  "test_patch": "diff --git a/tests/utils_tests/test_http.py b/tests/utils_tests/test_http.py\n--- a/tests/utils_tests/test_http.py\n+++ b/tests/utils_tests/test_http.py\n@@ -1,5 +1,6 @@\n import unittest\n from datetime import datetime\n+from unittest import mock\n \n from django.test import SimpleTestCase, ignore_warnings\n from django.utils.datastructures import MultiValueDict\n@@ -319,6 +320,14 @@ class HttpDateProcessingTests(unittest.TestCase):\n         parsed = parse_http_date('Sunday, 06-Nov-94 08:49:37 GMT')\n         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))\n \n+    def test_parsing_rfc850_at_next_century_boundary(self):\n+        current_time = datetime(2100, 11, 6, 8, 49, 37)\n+        with mock.patch('django.utils.http.datetime.datetime') as mocked_datetime:\n+            mocked_datetime.side_effect = datetime\n+            mocked_datetime.utcnow.return_value = current_time\n+            parsed = parse_http_date('Sunday, 06-Nov-50 08:49:37 GMT')\n+        self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(2150, 11, 6, 8, 49, 37))\n+\n     def test_parsing_asctime(self):\n         parsed = parse_http_date('Sun Nov  6 08:49:37 1994')\n         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))\n",
  "test_command": "python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_at_next_century_boundary"
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
FAIL: test_parsing_rfc850_at_next_century_boundary (utils_tests.test_http.HttpDateProcessingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_http.py", line 330, in test_parsing_rfc850_at_next_century_boundary
    self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(2150, 11, 6, 8, 49, 37))
AssertionError: datetime.datetime(2050, 11, 6, 8, 49, 37) != datetime.datetime(2150, 11, 6, 8, 49, 37)

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
