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
LEVEL_TAGS not updated when using @override_settings
Description
	
When reading messages inside tests, new message tags created using @override_settings is not updated.
That causes the django.contrib.messages.storage.base.Message.level_tag property results to be an empty string and not know the new tags.

</issue_statement>
<candidate_a_patch>
diff --git a/django/contrib/messages/storage/base.py b/django/contrib/messages/storage/base.py
index 01422066a6..a2ecd59bb2 100644
--- a/django/contrib/messages/storage/base.py
+++ b/django/contrib/messages/storage/base.py
@@ -1,9 +1,21 @@
 from django.conf import settings
 from django.contrib.messages import constants, utils
+from django.core.signals import setting_changed
+from django.dispatch import receiver
 
 LEVEL_TAGS = utils.get_level_tags()
 
 
+@receiver(setting_changed)
+def update_level_tags(setting, **kwargs):
+    """
+    Update LEVEL_TAGS when MESSAGE_TAGS setting is changed.
+    """
+    global LEVEL_TAGS
+    if setting == 'MESSAGE_TAGS':
+        LEVEL_TAGS = utils.get_level_tags()
+
+
 class Message:
     """
     Represent an actual message that can be stored in any of the supported

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/messages/storage/base.py b/django/contrib/messages/storage/base.py
index 01422066a6..3b5aeb6a07 100644
--- a/django/contrib/messages/storage/base.py
+++ b/django/contrib/messages/storage/base.py
@@ -1,9 +1,18 @@
 from django.conf import settings
 from django.contrib.messages import constants, utils
+from django.core.signals import setting_changed
+from django.dispatch import receiver
 
 LEVEL_TAGS = utils.get_level_tags()
 
 
+@receiver(setting_changed)
+def update_level_tags(**kwargs):
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
  "test_name": "test_update_level_tags_accepts_positional_setting",
  "specification_gap": "The MESSAGE_TAGS refresh hook must accept the changed setting name as an explicit positional argument, not only as a keyword supplied by override_settings.",
  "input_description": "Temporarily disconnect the automatic receiver, override MESSAGE_TAGS from INFO='outer' to INFO='inner', and directly invoke update_level_tags('MESSAGE_TAGS').",
  "expected_output": "A newly constructed INFO Message has the externally visible level_tag 'inner'. candidate_a accepts the positional setting argument and refreshes the tags; candidate_b raises TypeError.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises an arity variant of the refresh hook while asserting only the public Message.level_tag result. The nested overrides also restore global signal and tag state after the test.",
  "test_patch": "diff --git a/tests/messages_tests/tests.py b/tests/messages_tests/tests.py\n--- a/tests/messages_tests/tests.py\n+++ b/tests/messages_tests/tests.py\n@@ -1,8 +1,10 @@\n from unittest import mock\n \n from django.contrib.messages import constants\n+from django.contrib.messages.storage import base\n from django.contrib.messages.storage.base import Message\n-from django.test import SimpleTestCase\n+from django.core.signals import setting_changed\n+from django.test import SimpleTestCase, override_settings\n \n \n class MessageTests(SimpleTestCase):\n@@ -13,5 +15,18 @@ class MessageTests(SimpleTestCase):\n         self.assertEqual(msg_1, msg_1)\n         self.assertEqual(msg_1, mock.ANY)\n         self.assertNotEqual(msg_1, msg_2)\n         self.assertNotEqual(msg_1, msg_3)\n         self.assertNotEqual(msg_2, msg_3)\n+\n+    def test_update_level_tags_accepts_positional_setting(self):\n+        outer_tags = {constants.INFO: 'outer'}\n+        inner_tags = {constants.INFO: 'inner'}\n+        with override_settings(MESSAGE_TAGS=outer_tags):\n+            setting_changed.disconnect(base.update_level_tags)\n+            try:\n+                with override_settings(MESSAGE_TAGS=inner_tags):\n+                    base.update_level_tags('MESSAGE_TAGS')\n+                    message = Message(constants.INFO, 'message')\n+                    self.assertEqual(message.level_tag, 'inner')\n+            finally:\n+                setting_changed.connect(base.update_level_tags)\n",
  "test_command": "cd /testbed && python tests/runtests.py messages_tests.tests.MessageTests.test_update_level_tags_accepts_positional_setting"
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
      "duration_seconds": 1.386,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.372,
      "log_path": "02_execution/attempt_03/candidate_b.log"
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
E
======================================================================
ERROR: test_update_level_tags_accepts_positional_setting (messages_tests.tests.MessageTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/messages_tests/tests.py", line 28, in test_update_level_tags_accepts_positional_setting
    base.update_level_tags('MESSAGE_TAGS')
TypeError: update_level_tags() takes 0 positional arguments but 1 was given

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
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
  "duration_seconds": 1.356,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_update_level_tags_accepts_positional_setting"
  ],
  "required_any_substrings": [
    "FAILED"
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
  "reference_log_sha256": "b5b14521a003f286cb52061a035c9aa9c9efb2ab9af0e6c1b4cb1fc4b8763f3b"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: test_update_level_tags_accepts_positional_setting (messages_tests.tests.MessageTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/messages_tests/tests.py", line 25, in test_update_level_tags_accepts_positional_setting
    setting_changed.disconnect(base.update_level_tags)
AttributeError: module 'django.contrib.messages.storage.base' has no attribute 'update_level_tags'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
