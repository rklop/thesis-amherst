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
diff --git a/pylint/config/callback_actions.py b/pylint/config/callback_actions.py
index 76199539..747fb6ee 100644
--- a/pylint/config/callback_actions.py
+++ b/pylint/config/callback_actions.py
@@ -45,6 +45,33 @@ class _DoNothingAction(_CallbackAction):
     without erroring when they are then processed again by argparse.
     """
 
+    def __init__(
+        self,
+        option_strings: Sequence[str],
+        dest: str,
+        nargs: None = None,
+        const: None = None,
+        default: None = None,
+        type: None = None,
+        choices: None = None,
+        required: bool = False,
+        help: str = "",
+        metavar: str | tuple[str, str] = "",
+        **kwargs: Any,
+    ) -> None:
+        super().__init__(
+            option_strings,
+            dest,
+            0,
+            const,
+            default,
+            type,
+            choices,
+            required,
+            help,
+            metavar,
+        )
+
     def __call__(
         self,
         parser: argparse.ArgumentParser,
diff --git a/pylint/config/utils.py b/pylint/config/utils.py
index 8c790416..7ae9312b 100644
--- a/pylint/config/utils.py
+++ b/pylint/config/utils.py
@@ -207,6 +207,7 @@ PREPROCESSABLE_OPTIONS: dict[
     "--output": (True, _set_output),
     "--load-plugins": (True, _add_plugins),
     "--verbose": (False, _set_verbose_mode),
+    "-v": (False, _set_verbose_mode),
     "--enable-all-extensions": (False, _enable_all_extensions),
 }
 
@@ -218,7 +219,7 @@ def _preprocess_options(run: Run, args: Sequence[str]) -> list[str]:
     i = 0
     while i < len(args):
         argument = args[i]
-        if not argument.startswith("--"):
+        if not argument.startswith("--") and not argument.startswith("-"):
             processed_args.append(argument)
             i += 1
             continue

</candidate_a_patch>
<candidate_b_patch>
diff --git a/pylint/config/arguments_manager.py b/pylint/config/arguments_manager.py
index ea87de82..91412e58 100644
--- a/pylint/config/arguments_manager.py
+++ b/pylint/config/arguments_manager.py
@@ -39,6 +39,7 @@ from pylint.config.exceptions import (
 from pylint.config.help_formatter import _HelpFormatter
 from pylint.config.option import Option
 from pylint.config.option_parser import OptionParser
+from pylint.config.callback_actions import _DoNothingAction
 from pylint.config.options_provider_mixin import OptionsProviderMixIn
 from pylint.config.utils import _convert_option_to_argument, _parse_rich_type_value
 from pylint.constants import MAIN_CHECKER_NAME
@@ -213,12 +214,24 @@ class _ArgumentsManager:
                 help=argument.help,
             )
         elif isinstance(argument, _CallableArgument):
-            section_group.add_argument(
-                *argument.flags,
-                **argument.kwargs,
-                action=argument.action,
-                help=argument.help,
-            )
+            # The _DoNothingAction is used to allow pre-processing of certain options
+            # without erroring when they are then processed again by argparse.
+            # Set metavar='' to suppress the metavar in help message.
+            if argument.action == _DoNothingAction:
+                section_group.add_argument(
+                    *argument.flags,
+                    **argument.kwargs,
+                    action=argument.action,
+                    metavar="",
+                    help=argument.help,
+                )
+            else:
+                section_group.add_argument(
+                    *argument.flags,
+                    **argument.kwargs,
+                    action=argument.action,
+                    help=argument.help,
+                )
         elif isinstance(argument, _ExtendArgument):
             section_group.add_argument(
                 *argument.flags,
diff --git a/pylint/config/utils.py b/pylint/config/utils.py
index 8c790416..c0b862b4 100644
--- a/pylint/config/utils.py
+++ b/pylint/config/utils.py
@@ -207,6 +207,7 @@ PREPROCESSABLE_OPTIONS: dict[
     "--output": (True, _set_output),
     "--load-plugins": (True, _add_plugins),
     "--verbose": (False, _set_verbose_mode),
+    "-v": (False, _set_verbose_mode),
     "--enable-all-extensions": (False, _enable_all_extensions),
 }
 
@@ -218,7 +219,7 @@ def _preprocess_options(run: Run, args: Sequence[str]) -> list[str]:
     i = 0
     while i < len(args):
         argument = args[i]
-        if not argument.startswith("--"):
+        if not argument.startswith("-"):
             processed_args.append(argument)
             i += 1
             continue

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_short_verbose_does_not_consume_grouped_help_option",
  "specification_gap": "Because `-v` is a no-argument flag, it must not consume a following option in a standard grouped-short-option invocation. Thus `-vh` must parse like `-v -h`.",
  "input_description": "Invoke the public CLI as `python -m pylint -vh`, grouping the no-argument verbose and help short options.",
  "expected_output": "Pylint prints its help text, including `usage: pylint [options]`, and exits successfully with status 0.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This tests the behavioral arity of `-v`, not merely its displayed metavar. Candidate A makes the no-op action zero-arity, allowing argparse to process the grouped `-h`; candidate B leaves the action value-consuming, so it consumes `h` and subsequently exits with status 32 because no lint target or help action remains.",
  "test_patch": "diff --git a/tests/test_self.py b/tests/test_self.py\n--- a/tests/test_self.py\n+++ b/tests/test_self.py\n@@ -1294,6 +1294,18 @@ class TestCallbackOptions:\n         )\n         assert expected in process.stdout\n \n+    @staticmethod\n+    def test_short_verbose_does_not_consume_grouped_help_option() -> None:\n+        \"\"\"The no-argument -v flag should leave a grouped -h to be parsed.\"\"\"\n+        process = subprocess.run(\n+            [sys.executable, \"-m\", \"pylint\", \"-vh\"],\n+            capture_output=True,\n+            encoding=\"utf-8\",\n+            check=False,\n+        )\n+        assert process.returncode == 0\n+        assert \"usage: pylint [options]\" in process.stdout\n+\n     @staticmethod\n     def test_help_msg() -> None:\n         \"\"\"Test the --help-msg flag.\"\"\"\n",
  "test_command": "cd /testbed && python -m pytest -q tests/test_self.py::TestCallbackOptions::test_short_verbose_does_not_consume_grouped_help_option"
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
      "duration_seconds": 2.044,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 2.029,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
=============================== warnings summary ===============================
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

pylint/checkers/utils.py:455: 146 warnings
  /testbed/pylint/checkers/utils.py:455: DeprecationWarning: utils.check_messages will be removed in favour of calling utils.only_required_for_messages in pylint 3.0
    warnings.warn(

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
1 passed, 147 warnings in 0.40s
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_ TestCallbackOptions.test_short_verbose_does_not_consume_grouped_help_option __

    @staticmethod
    def test_short_verbose_does_not_consume_grouped_help_option() -> None:
        """The no-argument -v flag should leave a grouped -h to be parsed."""
        process = subprocess.run(
            [sys.executable, "-m", "pylint", "-vh"],
            capture_output=True,
            encoding="utf-8",
            check=False,
        )
>       assert process.returncode == 0
E       AssertionError: assert 32 == 0
E        +  where 32 = CompletedProcess(args=['/opt/miniconda3/envs/testbed/bin/python', '-m', 'pylint', '-vh'], returncode=32, stdout='usage...typing.get_type_hints``. Applies to Python versions\n                        3.7 - 3.9 (default: True)\n\n', stderr='').returncode

tests/test_self.py:1304: AssertionError
=============================== warnings summary ===============================
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

pylint/checkers/utils.py:455: 146 warnings
  /testbed/pylint/checkers/utils.py:455: DeprecationWarning: utils.check_messages will be removed in favour of calling utils.only_required_for_messages in pylint 3.0
    warnings.warn(

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED tests/test_self.py::TestCallbackOptions::test_short_verbose_does_not_consume_grouped_help_option
1 failed, 147 warnings in 0.45s
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/pylint/config/argument.py b/pylint/config/argument.py
--- a/pylint/config/argument.py
+++ b/pylint/config/argument.py
@@ -457,6 +457,7 @@ def __init__(
         kwargs: dict[str, Any],
         hide_help: bool,
         section: str | None,
+        metavar: str,
     ) -> None:
         super().__init__(
             flags=flags, arg_help=arg_help, hide_help=hide_help, section=section
@@ -467,3 +468,10 @@ def __init__(
 
         self.kwargs = kwargs
         """Any additional arguments passed to the action."""
+
+        self.metavar = metavar
+        """The metavar of the argument.
+
+        See:
+        https://docs.python.org/3/library/argparse.html#metavar
+        """
diff --git a/pylint/config/arguments_manager.py b/pylint/config/arguments_manager.py
--- a/pylint/config/arguments_manager.py
+++ b/pylint/config/arguments_manager.py
@@ -218,6 +218,7 @@ def _add_parser_option(
                 **argument.kwargs,
                 action=argument.action,
                 help=argument.help,
+                metavar=argument.metavar,
             )
         elif isinstance(argument, _ExtendArgument):
             section_group.add_argument(
diff --git a/pylint/config/utils.py b/pylint/config/utils.py
--- a/pylint/config/utils.py
+++ b/pylint/config/utils.py
@@ -71,6 +71,7 @@ def _convert_option_to_argument(
             kwargs=optdict.get("kwargs", {}),
             hide_help=optdict.get("hide", False),
             section=optdict.get("group", None),
+            metavar=optdict.get("metavar", None),
         )
     try:
         default = optdict["default"]
@@ -207,6 +208,7 @@ def _enable_all_extensions(run: Run, value: str | None) -> None:
     "--output": (True, _set_output),
     "--load-plugins": (True, _add_plugins),
     "--verbose": (False, _set_verbose_mode),
+    "-v": (False, _set_verbose_mode),
     "--enable-all-extensions": (False, _enable_all_extensions),
 }
 
@@ -218,7 +220,7 @@ def _preprocess_options(run: Run, args: Sequence[str]) -> list[str]:
     i = 0
     while i < len(args):
         argument = args[i]
-        if not argument.startswith("--"):
+        if not argument.startswith("-"):
             processed_args.append(argument)
             i += 1
             continue
diff --git a/pylint/lint/base_options.py b/pylint/lint/base_options.py
--- a/pylint/lint/base_options.py
+++ b/pylint/lint/base_options.py
@@ -544,6 +544,7 @@ def _make_run_options(self: Run) -> Options:
                 "help": "In verbose mode, extra non-checker-related info "
                 "will be displayed.",
                 "hide_from_config_file": True,
+                "metavar": "",
             },
         ),
         (
@@ -554,6 +555,7 @@ def _make_run_options(self: Run) -> Options:
                 "help": "Load and enable all available extensions. "
                 "Use --list-extensions to see a list all available extensions.",
                 "hide_from_config_file": True,
+                "metavar": "",
             },
         ),
         (

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 2.057,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_short_verbose_does_not_consume_grouped_help_option"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    " failed,"
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
  "reference_log_sha256": "c980ebb02d01b868fdd60c13aa75784958a14233bd44ea19de2600a646db5a2c"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_ TestCallbackOptions.test_short_verbose_does_not_consume_grouped_help_option __

    @staticmethod
    def test_short_verbose_does_not_consume_grouped_help_option() -> None:
        """The no-argument -v flag should leave a grouped -h to be parsed."""
        process = subprocess.run(
            [sys.executable, "-m", "pylint", "-vh"],
            capture_output=True,
            encoding="utf-8",
            check=False,
        )
>       assert process.returncode == 0
E       AssertionError: assert 32 == 0
E        +  where 32 = CompletedProcess(args=['/opt/miniconda3/envs/testbed/bin/python', '-m', 'pylint', '-vh'], returncode=32, stdout='usage...typing.get_type_hints``. Applies to Python versions\n                        3.7 - 3.9 (default: True)\n\n', stderr='').returncode

tests/test_self.py:1304: AssertionError
=============================== warnings summary ===============================
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/astroid/interpreter/_import/util.py:6: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

pylint/checkers/utils.py:455: 146 warnings
  /testbed/pylint/checkers/utils.py:455: DeprecationWarning: utils.check_messages will be removed in favour of calling utils.only_required_for_messages in pylint 3.0
    warnings.warn(

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED tests/test_self.py::TestCallbackOptions::test_short_verbose_does_not_consume_grouped_help_option
1 failed, 147 warnings in 0.45s
[pipeline] test_exit_code=1

</gold_execution_log>
