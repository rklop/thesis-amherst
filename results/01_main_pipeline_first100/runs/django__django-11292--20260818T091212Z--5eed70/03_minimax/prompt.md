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
Add --skip-checks option to management commands.
Description
	
Management commands already have skip_checks stealth option. I propose exposing this option on the command line. This would allow users to skip checks when running a command from the command line. Sometimes in a development environment, it is nice to move ahead with a task at hand rather than getting side tracked fixing a system check.

</issue_statement>

<original_test_patch>
diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py
--- a/tests/user_commands/tests.py
+++ b/tests/user_commands/tests.py
@@ -253,6 +253,16 @@ def test_disallowed_abbreviated_options(self):
         self.assertNoOutput(err)
         self.assertEqual(out.strip(), 'Set foo')
 
+    def test_skip_checks(self):
+        self.write_settings('settings.py', apps=['django.contrib.staticfiles', 'user_commands'], sdict={
+            # (staticfiles.E001) The STATICFILES_DIRS setting is not a tuple or
+            # list.
+            'STATICFILES_DIRS': '"foo"',
+        })
+        out, err = self.run_manage(['set_option', '--skip-checks', '--set', 'foo'])
+        self.assertNoOutput(err)
+        self.assertEqual(out.strip(), 'Set foo')
+
 
 class UtilsTests(SimpleTestCase):
 

</original_test_patch>

<candidate_a_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
index 0c3a981290..00fa3c5daf 100644
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -223,7 +223,7 @@ class BaseCommand:
     requires_system_checks = True
     # Arguments, common to all commands, which aren't defined by the argument
     # parser.
-    base_stealth_options = ('skip_checks', 'stderr', 'stdout')
+    base_stealth_options = ('stderr', 'stdout')
     # Command-specific options not defined by the argument parser.
     stealth_options = ()
 
@@ -286,6 +286,10 @@ class BaseCommand:
             '--force-color', action='store_true',
             help='Force colorization of the command output.',
         )
+        parser.add_argument(
+            '--skip-checks', action='store_true',
+            help='Skip system checks.',
+        )
         self.add_arguments(parser)
         return parser
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/core/management/base.py b/django/core/management/base.py
--- a/django/core/management/base.py
+++ b/django/core/management/base.py
@@ -95,7 +95,7 @@ class DjangoHelpFormatter(HelpFormatter):
     """
     show_last = {
         '--version', '--verbosity', '--traceback', '--settings', '--pythonpath',
-        '--no-color', '--force-color',
+        '--no-color', '--force-color', '--skip-checks',
     }
 
     def _reordered_actions(self, actions):
@@ -223,7 +223,7 @@ class BaseCommand:
     requires_system_checks = True
     # Arguments, common to all commands, which aren't defined by the argument
     # parser.
-    base_stealth_options = ('skip_checks', 'stderr', 'stdout')
+    base_stealth_options = ('stderr', 'stdout')
     # Command-specific options not defined by the argument parser.
     stealth_options = ()
 
@@ -286,6 +286,11 @@ def create_parser(self, prog_name, subcommand, **kwargs):
             '--force-color', action='store_true',
             help='Force colorization of the command output.',
         )
+        if self.requires_system_checks:
+            parser.add_argument(
+                '--skip-checks', action='store_true',
+                help='Skip system checks.',
+            )
         self.add_arguments(parser)
         return parser
 
@@ -357,7 +362,7 @@ def execute(self, *args, **options):
         if options.get('stderr'):
             self.stderr = OutputWrapper(options['stderr'])
 
-        if self.requires_system_checks and not options.get('skip_checks'):
+        if self.requires_system_checks and not options['skip_checks']:
             self.check()
         if self.requires_migrations_checks:
             self.check_migrations()

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_skip_checks_unavailable_without_automatic_checks",
  "specification_gap": "The candidates disagree on whether --skip-checks is exposed universally or only for commands that run automatic pre-execution system checks. The check command has requires_system_checks=False and performs checks explicitly, so accepting --skip-checks would silently accept a flag that cannot skip its work.",
  "input_description": "Call the public management API for the built-in check command with the CLI-style argument --skip-checks.",
  "expected_output": "Argument parsing raises CommandError with \"Error: unrecognized arguments: --skip-checks\".",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This boundary case tests the option's semantic invariant on an alternate public command. Candidate A exposes a misleading no-op flag on commands without automatic checks, while candidate B conditionally exposes it only where it can affect execution.",
  "test_patch": "diff --git a/tests/user_commands/tests.py b/tests/user_commands/tests.py\n--- a/tests/user_commands/tests.py\n+++ b/tests/user_commands/tests.py\n@@ -226,6 +226,11 @@ class CommandTests(SimpleTestCase):\n     def test_create_parser_kwargs(self):\n         \"\"\"BaseCommand.create_parser() passes kwargs to CommandParser.\"\"\"\n         epilog = 'some epilog text'\n         parser = BaseCommand().create_parser('prog_name', 'subcommand', epilog=epilog)\n         self.assertEqual(parser.epilog, epilog)\n+\n+    def test_skip_checks_unavailable_without_automatic_checks(self):\n+        msg = 'Error: unrecognized arguments: --skip-checks'\n+        with self.assertRaisesMessage(CommandError, msg):\n+            management.call_command('check', '--skip-checks')\n \n \n class CommandRunTests(AdminScriptTestCase):\n",
  "test_command": "cd /testbed && python tests/runtests.py user_commands.tests.CommandTests.test_skip_checks_unavailable_without_automatic_checks"
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
F
======================================================================
FAIL: test_skip_checks_unavailable_without_automatic_checks (user_commands.tests.CommandTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/user_commands/tests.py", line 235, in test_skip_checks_unavailable_without_automatic_checks
    management.call_command('check', '--skip-checks')
  File "/opt/miniconda3/envs/testbed/lib/python3.6/contextlib.py", line 88, in __exit__
    next(self.gen)
  File "/testbed/django/test/testcases.py", line 675, in _assert_raises_or_warns_cm
    yield cm
AssertionError: CommandError not raised

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
System check identified no issues (0 silenced).
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
