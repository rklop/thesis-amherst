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

eligible_failure_fingerprint=false. If false, do not accept.
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
  "test_name": "test_update_level_tags_rejects_positional_setting",
  "specification_gap": "The MESSAGE_TAGS refresh hook is a signal receiver whose payload is keyword-only. A positional `setting` argument must not be treated as a valid alternate calling convention.",
  "input_description": "Call the registered MESSAGE_TAGS update receiver directly with `'MESSAGE_TAGS'` as one positional argument.",
  "expected_output": "The call raises TypeError because the receiver accepts only keyword signal payloads. The supplied gold receiver uses only `**kwargs`; the generated candidate incorrectly accepts `setting` positionally and returns normally.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This arity boundary exposes the only semantic disagreement between the patches while clarifying that `setting` belongs to the signal's keyword payload rather than a positional receiver API. The diff was checked successfully with `git apply --check`.",
  "test_patch": "diff --git a/tests/messages_tests/tests.py b/tests/messages_tests/tests.py\nindex b49ccf84f4..0637b7cf8e 100644\n--- a/tests/messages_tests/tests.py\n+++ b/tests/messages_tests/tests.py\n@@ -1,7 +1,7 @@\n from unittest import mock\n \n from django.contrib.messages import constants\n-from django.contrib.messages.storage.base import Message\n+from django.contrib.messages.storage.base import Message, update_level_tags\n from django.test import SimpleTestCase\n \n \n@@ -15,3 +15,7 @@ class MessageTests(SimpleTestCase):\n         self.assertNotEqual(msg_1, msg_2)\n         self.assertNotEqual(msg_1, msg_3)\n         self.assertNotEqual(msg_2, msg_3)\n+\n+    def test_update_level_tags_rejects_positional_setting(self):\n+        with self.assertRaises(TypeError):\n+            update_level_tags('MESSAGE_TAGS')\n",
  "test_command": "python tests/runtests.py messages_tests.tests.MessageTests.test_update_level_tags_rejects_positional_setting"
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
      "duration_seconds": 1.392,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.346,
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
FAIL: test_update_level_tags_rejects_positional_setting (messages_tests.tests.MessageTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/messages_tests/tests.py", line 21, in test_update_level_tags_rejects_positional_setting
    update_level_tags('MESSAGE_TAGS')
AssertionError: TypeError not raised

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
  "duration_seconds": 1.395,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
null
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: tests (unittest.loader._FailedTest)
----------------------------------------------------------------------
ImportError: Failed to import test module: tests
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.9/unittest/loader.py", line 154, in loadTestsFromName
    module = __import__(module_name)
  File "/testbed/tests/messages_tests/tests.py", line 4, in <module>
    from django.contrib.messages.storage.base import Message, update_level_tags
ImportError: cannot import name 'update_level_tags' from 'django.contrib.messages.storage.base' (/testbed/django/contrib/messages/storage/base.py)


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
