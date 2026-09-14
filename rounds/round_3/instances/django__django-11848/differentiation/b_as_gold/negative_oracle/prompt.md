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
<candidate_a_patch>
diff --git a/django/utils/http.py b/django/utils/http.py
index 572cfb4347..8823f49514 100644
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
+            current_year = datetime.datetime.utcnow().year
+            year += (current_year // 100) * 100
+            if year > current_year + 50:
+                year -= 100
         month = MONTHS.index(m.group('mon').lower()) + 1
         day = int(m.group('day'))
         hour = int(m.group('hour'))

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/utils/http.py b/django/utils/http.py
index 572cfb4347..afe65fbdab 100644
--- a/django/utils/http.py
+++ b/django/utils/http.py
@@ -176,10 +176,12 @@ def parse_http_date(date):
     try:
         year = int(m.group('year'))
         if year < 100:
-            if year < 70:
-                year += 2000
-            else:
-                year += 1900
+            current_year = datetime.datetime.utcnow().year
+            year += 2000
+            if year - current_year > 50:
+                year -= 100
+            elif current_year - year >= 50:
+                year += 100
         month = MONTHS.index(m.group('mon').lower()) + 1
         day = int(m.group('day'))
         hour = int(m.group('hour'))

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_parsing_rfc850_year_exactly_50_years_in_future",
  "specification_gap": "A two-digit RFC 850 year exactly 50 years in the future must remain future-facing even when it crosses a century boundary; only dates more than 50 years ahead roll into the past.",
  "input_description": "Pin the current UTC date to 1969-01-01 and parse `Tuesday, 01-Jan-19 00:00:00 GMT`, representing the exact 50-year boundary.",
  "expected_output": "`parse_http_date()` returns a timestamp corresponding to 2019-01-01 00:00:00 UTC. Candidate_a instead resolves the year to 1919 by anchoring it to the current century.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This directly exercises the issue's strict \u201cmore than 50 years\u201d threshold across a century boundary. It checks public timestamp behavior under a deterministic clock without asserting implementation details.",
  "test_patch": "diff --git a/tests/utils_tests/test_http.py b/tests/utils_tests/test_http.py\n--- a/tests/utils_tests/test_http.py\n+++ b/tests/utils_tests/test_http.py\n@@ -1,5 +1,6 @@\n import unittest\n from datetime import datetime\n+from unittest import mock\n \n from django.test import SimpleTestCase, ignore_warnings\n from django.utils.datastructures import MultiValueDict\n@@ -319,7 +320,17 @@ class HttpDateProcessingTests(unittest.TestCase):\n     def test_parsing_rfc850(self):\n         parsed = parse_http_date('Sunday, 06-Nov-94 08:49:37 GMT')\n         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))\n \n+    def test_parsing_rfc850_year_exactly_50_years_in_future(self):\n+        class MockDateTime(datetime):\n+            @classmethod\n+            def utcnow(cls):\n+                return cls(1969, 1, 1)\n+\n+        with mock.patch('django.utils.http.datetime.datetime', MockDateTime):\n+            parsed = parse_http_date('Tuesday, 01-Jan-19 00:00:00 GMT')\n+        self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(2019, 1, 1))\n+\n     def test_parsing_asctime(self):\n         parsed = parse_http_date('Sun Nov  6 08:49:37 1994')\n         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))\n",
  "test_command": "cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_exactly_50_years_in_future"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.327,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.337,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_parsing_rfc850_year_exactly_50_years_in_future (utils_tests.test_http.HttpDateProcessingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_http.py", line 332, in test_parsing_rfc850_year_exactly_50_years_in_future
    self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(2019, 1, 1))
AssertionError: datetime.datetime(1919, 1, 1, 0, 0) != datetime.datetime(2019, 1, 1, 0, 0)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
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

</candidate_b_execution_log>
<official_gold_patch>
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

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.521,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_parsing_rfc850_year_exactly_50_years_in_future"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    "FAIL:"
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
  "reference_log_sha256": "9ced47d34fe3800c1e6bc6b2f16c681a33e9348e4653f1634c9ccad98ef7b68d"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_parsing_rfc850_year_exactly_50_years_in_future (utils_tests.test_http.HttpDateProcessingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_http.py", line 332, in test_parsing_rfc850_year_exactly_50_years_in_future
    self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(2019, 1, 1))
AssertionError: datetime.datetime(1919, 1, 1, 0, 0) != datetime.datetime(2019, 1, 1, 0, 0)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
[pipeline] test_exit_code=1

</gold_execution_log>
