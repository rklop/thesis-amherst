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

<original_test_patch>
diff --git a/tests/user_commands/management/commands/subparser_vanilla.py b/tests/user_commands/management/commands/subparser_vanilla.py
new file mode 100644
--- /dev/null
+++ b/tests/user_commands/management/commands/subparser_vanilla.py
@@ -0,0 +1,13 @@
+import argparse
+
+from django.core.management.base import BaseCommand
+
+
+class Command(BaseCommand):
+    def add_arguments(self, parser):
+        subparsers = parser.add_subparsers(parser_class=argparse.ArgumentParser)
+        parser_foo = subparsers.add_parser("foo")
+        parser_foo.add_argument("bar", type=int)
+
+    def handle(self, *args, **options):
+        pass
diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py
--- a/tests/user_commands/tests.py
+++ b/tests/user_commands/tests.py
@@ -468,6 +468,30 @@ def test_skip_checks(self):
         self.assertNoOutput(err)
         self.assertEqual(out.strip(), "Set foo")
 
+    def test_subparser_error_formatting(self):
+        self.write_settings("settings.py", apps=["user_commands"])
+        out, err = self.run_manage(["subparser", "foo", "twelve"])
+        self.maxDiff = None
+        self.assertNoOutput(out)
+        err_lines = err.splitlines()
+        self.assertEqual(len(err_lines), 2)
+        self.assertEqual(
+            err_lines[1],
+            "manage.py subparser foo: error: argument bar: invalid int value: 'twelve'",
+        )
+
+    def test_subparser_non_django_error_formatting(self):
+        self.write_settings("settings.py", apps=["user_commands"])
+        out, err = self.run_manage(["subparser_vanilla", "foo", "seven"])
+        self.assertNoOutput(out)
+        err_lines = err.splitlines()
+        self.assertEqual(len(err_lines), 2)
+        self.assertEqual(
+            err_lines[1],
+            "manage.py subparser_vanilla foo: error: argument bar: invalid int value: "
+            "'seven'",
+        )
+
 
 class UtilsTests(SimpleTestCase):
     def test_no_existent_external_program(self):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
index abc6f79a15..b08cb5ee25 100644
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -71,6 +71,25 @@ class CommandParser(ArgumentParser):
         else:
             raise CommandError("Error: %s" % message)
 
+    def add_subparsers(self, **kwargs):
+        parser_class = kwargs.get("parser_class", type(self))
+        # Capture the called_from_command_line value from this parser
+        # to pass it to any subparsers created
+        called_from_command_line = self.called_from_command_line
+
+        # Only wrap the parser_class if it's CommandParser or a subclass
+        if issubclass(parser_class, CommandParser):
+
+            class _CommandParserClass(parser_class):
+                def __init__(inner_self, **inner_kwargs):
+                    inner_kwargs.setdefault(
+                        "called_from_command_line", called_from_command_line
+                    )
+                    super().__init__(**inner_kwargs)
+
+            kwargs["parser_class"] = _CommandParserClass
+        return super().add_subparsers(**kwargs)
+
 
 def handle_default_options(options):
     """

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_subparser_custom_parser_class",
  "specification_gap": "The public `parser_class` extension point must instantiate the supplied `CommandParser` subclass directly while propagating `called_from_command_line`; it must not require that supplied class to support further subclassing.",
  "input_description": "A command-line `CommandParser` creates a `subcommand` parser using a custom `CommandParser` class that prohibits further subclassing. The child has a required positional `name`, and the root parser receives only `subcommand`.",
  "expected_output": "Parser construction succeeds. Parsing exits with status 2 and stderr contains `command subcommand: error: the following arguments are required: name`.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate B wraps the supplied parser class with `functools.partial`, so the exact class is instantiated with the inherited command-line flag. Candidate A dynamically subclasses it and therefore raises `TypeError` before parsing. This exercises argparse's public `parser_class` customization boundary while still verifying the user-visible error behavior from the issue.",
  "test_patch": "diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py\n--- a/tests/user_commands/tests.py\n+++ b/tests/user_commands/tests.py\n@@ -8,7 +8,8 @@ from admin_scripts.tests import AdminScriptTestCase\n from django.apps import apps\n from django.core import management\n from django.core.checks import Tags\n from django.core.management import BaseCommand, CommandError, find_commands\n+from django.core.management.base import CommandParser\n from django.core.management.utils import (\n     find_command,\n     get_random_secret_key,\n@@ -22,6 +23,28 @@ from django.test.utils import captured_stderr, extend_sys_path\n from django.utils import translation\n \n from .management.commands import dance\n \n \n+class CommandParserTests(SimpleTestCase):\n+    def test_subparser_custom_parser_class(self):\n+        class CustomParser(CommandParser):\n+            def __init_subclass__(cls, **kwargs):\n+                raise TypeError(\"CustomParser cannot be subclassed.\")\n+\n+        parser = CommandParser(\n+            prog=\"manage.py command\", called_from_command_line=True\n+        )\n+        subparsers = parser.add_subparsers(parser_class=CustomParser)\n+        subparser = subparsers.add_parser(\"subcommand\")\n+        subparser.add_argument(\"name\")\n+\n+        with captured_stderr() as stderr, self.assertRaises(SystemExit) as cm:\n+            parser.parse_args([\"subcommand\"])\n+        self.assertEqual(cm.exception.code, 2)\n+        self.assertIn(\n+            \"command subcommand: error: the following arguments are required: name\",\n+            stderr.getvalue(),\n+        )\n+\n+\n # A minimal set of apps to avoid system checks running on all apps.\n @override_settings(\n     INSTALLED_APPS=[\n",
  "test_command": "python tests/runtests.py user_commands.tests.CommandParserTests.test_subparser_custom_parser_class"
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
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_subparser_custom_parser_class (user_commands.tests.CommandParserTests.test_subparser_custom_parser_class)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/user_commands/tests.py", line 37, in test_subparser_custom_parser_class
    subparsers = parser.add_subparsers(parser_class=CustomParser)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/django/core/management/base.py", line 83, in add_subparsers
    class _CommandParserClass(parser_class):
  File "/testbed/tests/user_commands/tests.py", line 32, in __init_subclass__
    raise TypeError("CustomParser cannot be subclassed.")
TypeError: CustomParser cannot be subclassed.

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
