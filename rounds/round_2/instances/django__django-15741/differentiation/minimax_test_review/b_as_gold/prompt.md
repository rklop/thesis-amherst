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
diff --git a/django/utils/formats.py b/django/utils/formats.py
index 3f38322d84..0606cbf4ef 100644
--- a/django/utils/formats.py
+++ b/django/utils/formats.py
@@ -106,6 +106,7 @@ def get_format(format_type, lang=None, use_l10n=None):
     If use_l10n is provided and is not None, it forces the value to
     be localized (or not), overriding the value of settings.USE_L10N.
     """
+    format_type = str(format_type)
     if use_l10n is None:
         try:
             use_l10n = settings._USE_L10N_INTERNAL

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/utils/formats.py b/django/utils/formats.py
index 3f38322d84..7ca4834634 100644
--- a/django/utils/formats.py
+++ b/django/utils/formats.py
@@ -98,6 +98,7 @@ def get_format_modules(lang=None):
 
 
 def get_format(format_type, lang=None, use_l10n=None):
+    format_type = str(format_type)
     """
     For a specific format type, return the format for the current
     language (locale). Default to the format in the settings.

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_get_format_has_no_docstring",
  "specification_gap": "The authoritative patch places format_type coercion before the function's descriptive string literal. Consequently, that literal is no longer get_format's Python docstring and public introspection returns None.",
  "input_description": "Import django.utils.formats.get_format and inspect its __doc__ attribute.",
  "expected_output": "get_format.__doc__ is None.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Both patches behave identically for ordinary calls, so function metadata is their only runtime-observable semantic disagreement. This test captures that narrow distinction directly.",
  "test_patch": "diff --git a/tests/i18n/tests.py b/tests/i18n/tests.py\n--- a/tests/i18n/tests.py\n+++ b/tests/i18n/tests.py\n@@ -518,6 +518,9 @@ class TranslationThreadSafetyTests(SimpleTestCase):\n         self.assertLess(translation_count, len(trans_real._translations))\n \n \n class FormattingTests(SimpleTestCase):\n+    def test_get_format_has_no_docstring(self):\n+        self.assertIsNone(get_format.__doc__)\n+\n     def setUp(self):\n         super().setUp()\n         self.n = decimal.Decimal(\"66666.666\")\n",
  "test_command": "./tests/runtests.py i18n.tests.FormattingTests.test_get_format_has_no_docstring"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.403,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.399,
      "log_path": "02_execution/attempt_02/candidate_b.log"
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
FAIL: test_get_format_has_no_docstring (i18n.tests.FormattingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/i18n/tests.py", line 523, in test_get_format_has_no_docstring
    self.assertIsNone(get_format.__doc__)
AssertionError: "\n    For a specific format type, return the format for the current\n    language (locale). Default to the format in the settings.\n    format_type is the name of the format, e.g. 'DATE_FORMAT'.\n\n    If use_l10n is provided and is not None, it forces the value to\n    be localized (or not), overriding the value of settings.USE_L10N.\n    " is not None

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
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
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/utils/formats.py b/django/utils/formats.py
--- a/django/utils/formats.py
+++ b/django/utils/formats.py
@@ -113,6 +113,7 @@ def get_format(format_type, lang=None, use_l10n=None):
             use_l10n = settings.USE_L10N
     if use_l10n and lang is None:
         lang = get_language()
+    format_type = str(format_type)  # format_type may be lazy.
     cache_key = (format_type, lang)
     try:
         return _format_cache[cache_key]

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.413,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_get_format_has_no_docstring"
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
  "reference_log_sha256": "7c9657306c4680b591aa1a715af0e200cacf450380efe7c3fbd816ce171ca906"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_get_format_has_no_docstring (i18n.tests.FormattingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/i18n/tests.py", line 523, in test_get_format_has_no_docstring
    self.assertIsNone(get_format.__doc__)
AssertionError: "\n    For a specific format type, return the format for the current\n    language (locale). Default to the format in the settings.\n    format_type is the name of the format, e.g. 'DATE_FORMAT'.\n\n    If use_l10n is provided and is not None, it forces the value to\n    be localized (or not), overriding the value of settings.USE_L10N.\n    " is not None

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
