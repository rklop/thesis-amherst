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
call_command fails when argument of required mutually exclusive group is passed in kwargs.
Description
	
This error 
django.core.management.base.CommandError: Error: one of the arguments --shop-id --shop is required
is raised when I run 
call_command('my_command', shop_id=1)
the argument 'shop_id' is part of a required mutually exclusive group:
shop = parser.add_mutually_exclusive_group(required=True)
shop.add_argument('--shop-id', nargs='?', type=int, default=None, dest='shop_id')
shop.add_argument('--shop', nargs='?', type=str, default=None, dest='shop_name')
However, everything is fine when I call this command in this way:
call_command('my_command, '--shop-id=1')
In django sources I found that only those keyword arguments of call_command are passed to the parser that are defined as required:
# Any required arguments which are passed in via '**options' must be passed
# to parse_args().
parse_args += [
	'{}={}'.format(min(opt.option_strings), arg_options[opt.dest])
	for opt in parser._actions if opt.required and opt.dest in options
]
but in this special case both of them individually are not required, they are actually part of a group that is required. And the code of call_command does nothing with groups defined in the parser.

</issue_statement>

<original_test_patch>
diff --git a/tests/user_commands/management/commands/mutually_exclusive_required.py b/tests/user_commands/management/commands/mutually_exclusive_required.py
new file mode 100644
--- /dev/null
+++ b/tests/user_commands/management/commands/mutually_exclusive_required.py
@@ -0,0 +1,12 @@
+from django.core.management.base import BaseCommand
+
+
+class Command(BaseCommand):
+
+    def add_arguments(self, parser):
+        group = parser.add_mutually_exclusive_group(required=True)
+        group.add_argument('--foo-id', type=int, nargs='?', default=None)
+        group.add_argument('--foo-name', type=str, nargs='?', default=None)
+
+    def handle(self, *args, **options):
+        self.stdout.write(','.join(options))
diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py
--- a/tests/user_commands/tests.py
+++ b/tests/user_commands/tests.py
@@ -214,6 +214,16 @@ def test_command_add_arguments_after_common_arguments(self):
         management.call_command('common_args', stdout=out)
         self.assertIn('Detected that --version already exists', out.getvalue())
 
+    def test_mutually_exclusive_group_required_options(self):
+        out = StringIO()
+        management.call_command('mutually_exclusive_required', foo_id=1, stdout=out)
+        self.assertIn('foo_id', out.getvalue())
+        management.call_command('mutually_exclusive_required', foo_name='foo', stdout=out)
+        self.assertIn('foo_name', out.getvalue())
+        msg = 'Error: one of the arguments --foo-id --foo-name is required'
+        with self.assertRaisesMessage(CommandError, msg):
+            management.call_command('mutually_exclusive_required', stdout=out)
+
     def test_subparser(self):
         out = StringIO()
         management.call_command('subparser', 'foo', 12, stdout=out)

</original_test_patch>

<candidate_a_patch>
diff --git a/django/core/management/__init__.py b/django/core/management/__init__.py
index 5f2e64761e..25993feb70 100644
--- a/django/core/management/__init__.py
+++ b/django/core/management/__init__.py
@@ -136,6 +136,14 @@ def call_command(command_name, *args, **options):
         '{}={}'.format(min(opt.option_strings), arg_options[opt.dest])
         for opt in parser_actions if opt.required and opt.dest in options
     ]
+    # Also handle options that are part of a required mutually exclusive group.
+    for group in parser._mutually_exclusive_groups:
+        if group.required:
+            for action in group._group_actions:
+                if action.dest in options:
+                    parse_args.append(
+                        '{}={}'.format(min(action.option_strings), arg_options[action.dest])
+                    )
     defaults = parser.parse_args(args=parse_args)
     defaults = dict(defaults._get_kwargs(), **arg_options)
     # Raise an error if any unknown options were passed.

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/core/management/__init__.py b/django/core/management/__init__.py
--- a/django/core/management/__init__.py
+++ b/django/core/management/__init__.py
@@ -130,11 +130,19 @@ def get_actions(parser):
                 yield opt
 
     parser_actions = list(get_actions(parser))
+    mutually_exclusive_required_options = {
+        opt
+        for group in parser._mutually_exclusive_groups
+        for opt in group._group_actions if group.required
+    }
     # Any required arguments which are passed in via **options must be passed
     # to parse_args().
     parse_args += [
         '{}={}'.format(min(opt.option_strings), arg_options[opt.dest])
-        for opt in parser_actions if opt.required and opt.dest in options
+        for opt in parser_actions if (
+            opt.dest in options and
+            (opt.required or opt in mutually_exclusive_required_options)
+        )
     ]
     defaults = parser.parse_args(args=parse_args)
     defaults = dict(defaults._get_kwargs(), **arg_options)

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_call_command_required_group_preserves_action_order",
  "specification_gap": "call_command() should preserve the parser-visible ordering of options when translating keyword arguments, including options from required mutually exclusive groups. Candidate A appends ordinary required options first and group options afterward, reordering custom Action execution; candidate B keeps parser action order.",
  "input_description": "A command declares a required mutually exclusive option before an ordinary required option. A public argparse.Action records invocation order. The command is invoked once with ordered CLI arguments and once with the equivalent ordered keyword arguments.",
  "expected_output": "Both public entry points return exactly \"group_value,required_value\". The keyword invocation must be observationally equivalent to the CLI invocation.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests an alternate public entry point and argparse's custom Action extension point. Candidate A returns \"required_value,group_value\" for kwargs because it partitions required actions by category; candidate B preserves declaration order.",
  "test_patch": "diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py\n--- a/tests/user_commands/tests.py\n+++ b/tests/user_commands/tests.py\n@@ -1,3 +1,4 @@\n+import argparse\n import os\n from io import StringIO\n from unittest import mock\n@@ -206,7 +207,44 @@ class CommandTests(SimpleTestCase):\n         out = StringIO()\n         management.call_command('required_option', '--need-me=foo', needme2='bar', stdout=out)\n         self.assertIn('need_me', out.getvalue())\n         self.assertIn('needme2', out.getvalue())\n \n+    def test_call_command_required_group_preserves_action_order(self):\n+        class RecordingAction(argparse.Action):\n+            def __call__(self, parser, namespace, value, option_string=None):\n+                trace = getattr(namespace, 'action_trace', ())\n+                namespace.action_trace = trace + (self.dest,)\n+                setattr(namespace, self.dest, value)\n+\n+        class Command(BaseCommand):\n+            requires_system_checks = False\n+\n+            def add_arguments(self, parser):\n+                group = parser.add_mutually_exclusive_group(required=True)\n+                group.add_argument(\n+                    '--group-value',\n+                    action=RecordingAction,\n+                )\n+                parser.add_argument(\n+                    '--required-value',\n+                    action=RecordingAction,\n+                    required=True,\n+                )\n+\n+            def handle(self, *args, **options):\n+                return ','.join(options['action_trace'])\n+\n+        command = Command()\n+        positional_result = management.call_command(\n+            command, '--group-value=group', '--required-value=required',\n+            stdout=StringIO(),\n+        )\n+        keyword_result = management.call_command(\n+            command, group_value='group', required_value='required',\n+            stdout=StringIO(),\n+        )\n+        self.assertEqual(keyword_result, positional_result)\n+        self.assertEqual(keyword_result, 'group_value,required_value')\n+\n     def test_command_add_arguments_after_common_arguments(self):\n         out = StringIO()\n         management.call_command('common_args', stdout=out)\n",
  "test_command": "python tests/runtests.py user_commands.tests.CommandTests.test_call_command_required_group_preserves_action_order"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_call_command_required_group_preserves_action_order (user_commands.tests.CommandTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/user_commands/tests.py", line 247, in test_call_command_required_group_preserves_action_order
    self.assertEqual(keyword_result, positional_result)
AssertionError: 'required_value,group_value' != 'group_value,required_value'
- required_value,group_value
+ group_value,required_value


----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
