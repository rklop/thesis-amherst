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
Management command subparsers don’t retain error formatting
Description
	
Django management commands use a subclass of argparse.ArgumentParser, CommandParser, that takes some extra arguments to improve error formatting. These arguments are not copied into subparsers, created via CommandParser.add_subparsers().add_parser(). Missing arguments to subparsers thus end as stack traces on the CLI, rather than human-facing usage messages.
For example take this command with a subparser:
from django.core.management.base import BaseCommand
class Command(BaseCommand):
	def add_arguments(self, parser):
		subparsers = parser.add_subparsers(required=True)
		create = subparsers.add_parser("create")
		create.add_argument("name")
	def handle(self, *args, **options):
		pass
Missing the required subparser name argument gives the usage message, as for any normal argument:
$ ./manage.py cheeses
usage: manage.py cheeses [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS] [--pythonpath PYTHONPATH] [--traceback] [--no-color] [--force-color] [--skip-checks] {create} ...
manage.py cheeses: error: the following arguments are required: {create}
But missing the name argument to create fails with a stacktrace:
$ ./manage.py cheeses create
Traceback (most recent call last):
 File "/Users/chainz/tmp/subparserstest/./manage.py", line 21, in <module>
	main()
...
 File "/Users/chainz/.pyenv/versions/3.11.0/lib/python3.11/argparse.py", line 2131, in _parse_known_args
	self.error(_('the following arguments are required: %s') %
 File "/Users/chainz/Documents/Projects/django/django/core/management/base.py", line 72, in error
	raise CommandError("Error: %s" % message)
django.core.management.base.CommandError: Error: the following arguments are required: name
We can correct this by ensuring that the subparser action returned by add_subparsers() copies the relevant arguments through to constructed subparsers.
(Originally reported by Mark Gregson on django-developers: ​https://groups.google.com/g/django-developers/c/oWcaxkxQ-KI/m/4NUhLjddBwAJ )

</issue_statement>
<candidate_a_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
index abc6f79a15..dd1a404e4b 100644
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -3,6 +3,7 @@ Base classes for writing management commands (named commands which can
 be executed through ``django-admin`` or ``manage.py``).
 """
 import argparse
+from functools import partial
 import os
 import sys
 from argparse import ArgumentParser, HelpFormatter
@@ -71,6 +72,15 @@ class CommandParser(ArgumentParser):
         else:
             raise CommandError("Error: %s" % message)
 
+    def add_subparsers(self, **kwargs):
+        parser_class = kwargs.get("parser_class", type(self))
+        if issubclass(parser_class, CommandParser):
+            kwargs["parser_class"] = partial(
+                parser_class,
+                called_from_command_line=self.called_from_command_line,
+            )
+        return super().add_subparsers(**kwargs)
+
 
 def handle_default_options(options):
     """

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
index abc6f79a15..d7e1e82cfc 100644
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -71,6 +71,26 @@ class CommandParser(ArgumentParser):
         else:
             raise CommandError("Error: %s" % message)
 
+    def add_subparsers(self, **kwargs):
+        parser_class = kwargs.get("parser_class", type(self))
+        if issubclass(parser_class, CommandParser):
+            kwargs["parser_class"] = self._make_subparser_class(parser_class)
+        return super().add_subparsers(**kwargs)
+
+    def _make_subparser_class(self, parser_class):
+        """Create a subparser class that inherits settings from the parent."""
+        called_from_command_line = self.called_from_command_line
+
+        class Subparser(parser_class):
+            def __init__(self, **kwargs):
+                # Only pass called_from_command_line if explicitly provided by user
+                # and not already set in kwargs
+                if called_from_command_line is not None and "called_from_command_line" not in kwargs:
+                    kwargs["called_from_command_line"] = called_from_command_line
+                super().__init__(**kwargs)
+
+        return Subparser
+
 
 def handle_default_options(options):
     """

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_subparser_parser_class_preserves_default",
  "specification_gap": "When a parent CommandParser leaves called_from_command_line unspecified (None), creating a subparser with a custom CommandParser subclass must preserve that subclass's own default instead of explicitly overriding it with None.",
  "input_description": "Create a CommandParser whose CLI-origin state is unspecified, configure add_subparsers() with a custom CommandParser subclass that defaults to command-line error handling, add a required positional argument to the child parser, and parse only the child command name.",
  "expected_output": "Parsing exits with status 2 and writes a human-facing argparse error containing \"manage.py command child: error: the following arguments are required: name\" to stderr, rather than raising CommandError with a traceback-producing path.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the public parser_class extension point and the boundary between an unspecified parent value and a custom subclass default. The gold patch conditionally inherits only non-None state, while the generated candidate always binds the parent's None value and suppresses the subclass's CLI behavior.",
  "test_patch": "diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py\n--- a/tests/user_commands/tests.py\n+++ b/tests/user_commands/tests.py\n@@ -405,6 +405,29 @@ class CommandTests(SimpleTestCase):\n         msg = \"Error: the following arguments are required: subcommand\"\n         with self.assertRaisesMessage(CommandError, msg):\n             management.call_command(\"subparser_dest\", subcommand=\"foo\", bar=12)\n \n+    def test_subparser_parser_class_preserves_default(self):\n+        class CliCommandParser(management.CommandParser):\n+            def __init__(\n+                self, *, called_from_command_line=True, **kwargs\n+            ):\n+                super().__init__(\n+                    called_from_command_line=called_from_command_line, **kwargs\n+                )\n+\n+        parser = management.CommandParser(prog=\"manage.py command\")\n+        subparsers = parser.add_subparsers(parser_class=CliCommandParser)\n+        child_parser = subparsers.add_parser(\"child\")\n+        child_parser.add_argument(\"name\")\n+\n+        with captured_stderr() as stderr, self.assertRaises(SystemExit) as cm:\n+            parser.parse_args([\"child\"])\n+        self.assertEqual(cm.exception.code, 2)\n+        self.assertIn(\n+            \"manage.py command child: error: \"\n+            \"the following arguments are required: name\",\n+            stderr.getvalue(),\n+        )\n+\n     def test_create_parser_kwargs(self):\n         \"\"\"BaseCommand.create_parser() passes kwargs to CommandParser.\"\"\"\n         epilog = \"some epilog text\"\n",
  "test_command": "cd /testbed && python tests/runtests.py user_commands.tests.CommandTests.test_subparser_parser_class_preserves_default"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 3,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.532,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.412,
      "log_path": "02_execution/attempt_03/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_subparser_parser_class_preserves_default (user_commands.tests.CommandTests.test_subparser_parser_class_preserves_default)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/user_commands/tests.py", line 424, in test_subparser_parser_class_preserves_default
    parser.parse_args(["child"])
  File "/testbed/django/core/management/base.py", line 67, in parse_args
    return super().parse_args(args, namespace)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1874, in parse_args
    args, argv = self.parse_known_args(args, namespace)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1907, in parse_known_args
    namespace, args = self._parse_known_args(args, namespace)
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2131, in _parse_known_args
    stop_index = consume_positionals(start_index)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2087, in consume_positionals
    take_action(action, args)
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1983, in take_action
    action(self, namespace, argument_values, option_string)
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1246, in __call__
    subnamespace, arg_strings = parser.parse_known_args(arg_strings, None)
                                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1907, in parse_known_args
    namespace, args = self._parse_known_args(args, namespace)
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2156, in _parse_known_args
    self.error(_('the following arguments are required: %s') %
  File "/testbed/django/core/management/base.py", line 73, in error
    raise CommandError("Error: %s" % message)
django.core.management.base.CommandError: Error: the following arguments are required: name

----------------------------------------------------------------------
Ran 1 test in 0.003s

FAILED (errors=1)
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.003s

OK
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -6,6 +6,7 @@
 import os
 import sys
 from argparse import ArgumentParser, HelpFormatter
+from functools import partial
 from io import TextIOBase
 
 import django
@@ -71,6 +72,15 @@ def error(self, message):
         else:
             raise CommandError("Error: %s" % message)
 
+    def add_subparsers(self, **kwargs):
+        parser_class = kwargs.get("parser_class", type(self))
+        if issubclass(parser_class, CommandParser):
+            kwargs["parser_class"] = partial(
+                parser_class,
+                called_from_command_line=self.called_from_command_line,
+            )
+        return super().add_subparsers(**kwargs)
+
 
 def handle_default_options(options):
     """

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.451,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_subparser_parser_class_preserves_default"
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
  "reference_log_sha256": "5c11db4fd05efe90672208c122465da39be7ca5aa83d111805d7b8694f10ecfd"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_subparser_parser_class_preserves_default (user_commands.tests.CommandTests.test_subparser_parser_class_preserves_default)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/user_commands/tests.py", line 424, in test_subparser_parser_class_preserves_default
    parser.parse_args(["child"])
  File "/testbed/django/core/management/base.py", line 67, in parse_args
    return super().parse_args(args, namespace)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1874, in parse_args
    args, argv = self.parse_known_args(args, namespace)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1907, in parse_known_args
    namespace, args = self._parse_known_args(args, namespace)
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2131, in _parse_known_args
    stop_index = consume_positionals(start_index)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2087, in consume_positionals
    take_action(action, args)
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1983, in take_action
    action(self, namespace, argument_values, option_string)
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1246, in __call__
    subnamespace, arg_strings = parser.parse_known_args(arg_strings, None)
                                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 1907, in parse_known_args
    namespace, args = self._parse_known_args(args, namespace)
                      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/opt/miniconda3/envs/testbed/lib/python3.11/argparse.py", line 2156, in _parse_known_args
    self.error(_('the following arguments are required: %s') %
  File "/testbed/django/core/management/base.py", line 73, in error
    raise CommandError("Error: %s" % message)
django.core.management.base.CommandError: Error: the following arguments are required: name

----------------------------------------------------------------------
Ran 1 test in 0.003s

FAILED (errors=1)
[pipeline] test_exit_code=1

</gold_execution_log>
