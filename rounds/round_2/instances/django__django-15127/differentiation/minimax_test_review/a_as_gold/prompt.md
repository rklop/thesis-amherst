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
diff --git a/django/contrib/messages/storage/base.py b/django/contrib/messages/storage/base.py
index 01422066a6..d5d38701e1 100644
--- a/django/contrib/messages/storage/base.py
+++ b/django/contrib/messages/storage/base.py
@@ -1,7 +1,48 @@
 from django.conf import settings
 from django.contrib.messages import constants, utils
 
-LEVEL_TAGS = utils.get_level_tags()
+
+class LEVEL_TAGSdict(dict):
+    """
+    A dict subclass that retrieves the current level tags from settings
+    each time it's accessed. This ensures that @override_settings works
+    correctly with message level tags.
+    """
+    def __getitem__(self, key):
+        return utils.get_level_tags().__getitem__(key)
+
+    def __contains__(self, key):
+        return utils.get_level_tags().__contains__(key)
+
+    def __iter__(self):
+        return iter(utils.get_level_tags())
+
+    def __len__(self):
+        return len(utils.get_level_tags())
+
+    def __repr__(self):
+        return utils.get_level_tags().__repr__()
+
+    def __eq__(self, other):
+        return utils.get_level_tags() == other
+
+    def keys(self):
+        return utils.get_level_tags().keys()
+
+    def values(self):
+        return utils.get_level_tags().values()
+
+    def items(self):
+        return utils.get_level_tags().items()
+
+    def get(self, key, default=None):
+        return utils.get_level_tags().get(key, default)
+
+    def copy(self):
+        return utils.get_level_tags().copy()
+
+
+LEVEL_TAGS = LEVEL_TAGSdict()
 
 
 class Message:

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/messages/storage/base.py b/django/contrib/messages/storage/base.py
index 01422066a6..8c4d14d869 100644
--- a/django/contrib/messages/storage/base.py
+++ b/django/contrib/messages/storage/base.py
@@ -1,9 +1,21 @@
 from django.conf import settings
 from django.contrib.messages import constants, utils
+from django.core.signals import setting_changed
+from django.dispatch import receiver
 
 LEVEL_TAGS = utils.get_level_tags()
 
 
+@receiver(setting_changed)
+def update_level_tags(**kwargs):
+    """
+    Update LEVEL_TAGS when MESSAGE_TAGS setting changes.
+    """
+    if kwargs['setting'] == 'MESSAGE_TAGS':
+        global LEVEL_TAGS
+        LEVEL_TAGS = utils.get_level_tags()
+
+
 class Message:
     """
     Represent an actual message that can be stored in any of the supported

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_imported_level_tags_track_override",
  "specification_gap": "A LEVEL_TAGS reference imported before override_settings() should expose the active MESSAGE_TAGS values. Rebinding only the storage module attribute leaves such references stale.",
  "input_description": "Import LEVEL_TAGS, then override MESSAGE_TAGS so the INFO level maps to 'custom-info', and access INFO through the imported mapping.",
  "expected_output": "LEVEL_TAGS[constants.INFO] returns 'custom-info' while the override is active.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This covers an alternate public import path for the constant named in the issue. Candidate_a keeps the imported mapping settings-aware, while candidate_b replaces the module binding and leaves the previously imported mapping unchanged.",
  "test_patch": "diff --git a/tests/messages_tests/tests.py b/tests/messages_tests/tests.py\n--- a/tests/messages_tests/tests.py\n+++ b/tests/messages_tests/tests.py\n@@ -1,8 +1,8 @@\n from unittest import mock\n \n from django.contrib.messages import constants\n-from django.contrib.messages.storage.base import Message\n-from django.test import SimpleTestCase\n+from django.contrib.messages.storage.base import LEVEL_TAGS, Message\n+from django.test import SimpleTestCase, override_settings\n \n \n class MessageTests(SimpleTestCase):\n@@ -15,3 +15,7 @@ class MessageTests(SimpleTestCase):\n         self.assertNotEqual(msg_1, msg_2)\n         self.assertNotEqual(msg_1, msg_3)\n         self.assertNotEqual(msg_2, msg_3)\n+\n+    @override_settings(MESSAGE_TAGS={constants.INFO: 'custom-info'})\n+    def test_imported_level_tags_track_override(self):\n+        self.assertEqual(LEVEL_TAGS[constants.INFO], 'custom-info')\n",
  "test_command": "cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_imported_level_tags_track_override"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.422,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.364,
      "log_path": "02_execution/attempt_02/candidate_b.log"
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
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_imported_level_tags_track_override (messages_tests.tests.MessageTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 437, in inner
    return func(*args, **kwargs)
  File "/testbed/tests/messages_tests/tests.py", line 21, in test_imported_level_tags_track_override
    self.assertEqual(LEVEL_TAGS[constants.INFO], 'custom-info')
AssertionError: 'info' != 'custom-info'
- info
+ custom-info


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/contrib/messages/apps.py b/django/contrib/messages/apps.py
--- a/django/contrib/messages/apps.py
+++ b/django/contrib/messages/apps.py
@@ -1,7 +1,18 @@
 from django.apps import AppConfig
+from django.contrib.messages.storage import base
+from django.contrib.messages.utils import get_level_tags
+from django.test.signals import setting_changed
 from django.utils.translation import gettext_lazy as _
 
 
+def update_level_tags(setting, **kwargs):
+    if setting == 'MESSAGE_TAGS':
+        base.LEVEL_TAGS = get_level_tags()
+
+
 class MessagesConfig(AppConfig):
     name = 'django.contrib.messages'
     verbose_name = _("Messages")
+
+    def ready(self):
+        setting_changed.connect(update_level_tags)

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.51,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_imported_level_tags_track_override"
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
  "reference_log_sha256": "ff8f7f08ecb8a83da374a4e3260c0fc57ac5d4a251fa37839150d0bc4c84dd61"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_imported_level_tags_track_override (messages_tests.tests.MessageTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 437, in inner
    return func(*args, **kwargs)
  File "/testbed/tests/messages_tests/tests.py", line 21, in test_imported_level_tags_track_override
    self.assertEqual(LEVEL_TAGS[constants.INFO], 'custom-info')
AssertionError: 'info' != 'custom-info'
- info
+ custom-info


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
