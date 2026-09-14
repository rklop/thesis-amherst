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
diff --git a/django/utils/http.py b/django/utils/http.py
index 572cfb4347..a472e5d2a9 100644
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
+            year += 2000
+            if year > current_year + 50:
+                year -= 100
         month = MONTHS.index(m.group('mon').lower()) + 1
         day = int(m.group('day'))
         hour = int(m.group('hour'))

</candidate_a_patch>
<candidate_b_patch>
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

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_parsing_rfc850_fifty_year_century_boundary",
  "specification_gap": "Rollback applies only when an RFC 850 year is more than 50 years in the future. The generated candidate incorrectly maps `20` to 1920 when the current year is 1970, although 2020 is exactly 50 years ahead and must remain unchanged.",
  "input_description": "Patch the parser's UTC clock to 1970-01-01 and parse `Friday, 06-Nov-20 08:49:37 GMT`.",
  "expected_output": "`parse_http_date()` returns an epoch value representing 2020-11-06 08:49:37 UTC, not 1920.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This tests the strict 50-year boundary across a century transition through the public parser, exposing the generated candidate's observable century-selection error without asserting implementation details.",
  "test_patch": "diff --git a/tests/utils_tests/test_http.py b/tests/utils_tests/test_http.py\n--- a/tests/utils_tests/test_http.py\n+++ b/tests/utils_tests/test_http.py\n@@ -1,5 +1,6 @@\n import unittest\n from datetime import datetime\n+from unittest import mock\n \n from django.test import SimpleTestCase, ignore_warnings\n from django.utils.datastructures import MultiValueDict\n@@ -319,6 +320,19 @@ class HttpDateProcessingTests(unittest.TestCase):\n     def test_parsing_rfc850(self):\n         parsed = parse_http_date('Sunday, 06-Nov-94 08:49:37 GMT')\n         self.assertEqual(datetime.utcfromtimestamp(parsed), datetime(1994, 11, 6, 8, 49, 37))\n+\n+    def test_parsing_rfc850_fifty_year_century_boundary(self):\n+        class MockDateTime(datetime):\n+            @classmethod\n+            def utcnow(cls):\n+                return cls(1970, 1, 1)\n+\n+        with mock.patch('django.utils.http.datetime.datetime', MockDateTime):\n+            parsed = parse_http_date('Friday, 06-Nov-20 08:49:37 GMT')\n+        self.assertEqual(\n+            datetime.utcfromtimestamp(parsed),\n+            datetime(2020, 11, 6, 8, 49, 37),\n+        )\n \n     def test_parsing_asctime(self):\n         parsed = parse_http_date('Sun Nov  6 08:49:37 1994')\n",
  "test_command": "cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_fifty_year_century_boundary"
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
      "duration_seconds": 1.342,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.465,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
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

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_parsing_rfc850_fifty_year_century_boundary (utils_tests.test_http.HttpDateProcessingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_http.py", line 334, in test_parsing_rfc850_fifty_year_century_boundary
    datetime(2020, 11, 6, 8, 49, 37),
AssertionError: datetime.datetime(1920, 11, 6, 8, 49, 37) != datetime.datetime(2020, 11, 6, 8, 49, 37)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

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
  "duration_seconds": 1.4,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_parsing_rfc850_fifty_year_century_boundary"
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
  "reference_log_sha256": "b5409e8b828937ec5a5eef6022cfef71f385b14cb8611c7b74e37381a1d7061b"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_parsing_rfc850_fifty_year_century_boundary (utils_tests.test_http.HttpDateProcessingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/utils_tests/test_http.py", line 334, in test_parsing_rfc850_fifty_year_century_boundary
    datetime(2020, 11, 6, 8, 49, 37),
AssertionError: datetime.datetime(1920, 11, 6, 8, 49, 37) != datetime.datetime(2020, 11, 6, 8, 49, 37)

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
[pipeline] test_exit_code=1

</gold_execution_log>
