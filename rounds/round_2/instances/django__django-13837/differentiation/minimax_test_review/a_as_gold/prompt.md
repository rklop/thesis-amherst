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
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
index b8efb9f881..f2fec399fd 100644
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -217,11 +217,20 @@ def get_child_arguments():
     on reloading.
     """
     import django.__main__
+    import __main__
     django_main_path = Path(django.__main__.__file__)
     py_script = Path(sys.argv[0])
 
     args = [sys.executable] + ['-W%s' % o for o in sys.warnoptions]
-    if py_script == django_main_path:
+    # __spec__ is set when the server was started with the `-m` option,
+    # see https://docs.python.org/3/reference/import.html#main-spec
+    # __spec__ may not exist, e.g. when running in a Conda env.
+    if getattr(__main__, '__spec__', None) is not None and __main__.__spec__.parent:
+        # The server was started with `python -m django runserver`
+        # or `python -m package.name runserver`.
+        args += ['-m', __main__.__spec__.parent]
+        args += sys.argv[1:]
+    elif py_script == django_main_path:
         # The server was started with `python -m django runserver`.
         args += ['-m', 'django']
         args += sys.argv[1:]

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
index b8efb9f881..a559d7db43 100644
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -216,12 +216,20 @@ def get_child_arguments():
     executable is reported to not have the .exe extension which can cause bugs
     on reloading.
     """
+    import __main__
     import django.__main__
     django_main_path = Path(django.__main__.__file__)
     py_script = Path(sys.argv[0])
 
     args = [sys.executable] + ['-W%s' % o for o in sys.warnoptions]
-    if py_script == django_main_path:
+    # __spec__ is set when the server was started with the `-m` option,
+    # see https://docs.python.org/3/reference/import.html#main-spec
+    # __spec__ may not exist, e.g. when running in a Conda env.
+    if getattr(__main__, '__spec__', None) is not None and __main__.__spec__.parent:
+        # The server was started with `python -m pkg runserver`.
+        args += ['-m', __main__.__spec__.parent]
+        args += sys.argv[1:]
+    elif py_script == django_main_path:
         # The server was started with `python -m django runserver`.
         args += ['-m', 'django']
         args += sys.argv[1:]

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_main_module_is_resolved_after_django_main",
  "specification_gap": "get_child_arguments() must inspect the currently registered top-level __main__ module after Django's CLI module has been imported, rather than retaining a module object captured before that import.",
  "input_description": "Call get_child_arguments() with an existing script path while importing django.__main__ changes sys.modules['__main__'] from an entry point whose spec parent is 'stale.package' to one whose spec parent is 'pkg_other_than_django'.",
  "expected_output": "The returned child command is [sys.executable, '-m', 'pkg_other_than_django', 'runserver'], reflecting the active entry-point module.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises the sole runtime disagreement between the patches: candidate_a resolves __main__ after importing django.__main__, while candidate_b retains the earlier module and produces '-m stale.package'. The corrected unified diff passes git apply --check against the pristine repository.",
  "test_patch": "diff --git a/tests/utils_tests/test_autoreload.py b/tests/utils_tests/test_autoreload.py\n--- a/tests/utils_tests/test_autoreload.py\n+++ b/tests/utils_tests/test_autoreload.py\n@@ -159,11 +159,34 @@ class TestChildArguments(SimpleTestCase):\n     @mock.patch('sys.argv', [django.__main__.__file__, 'runserver'])\n     @mock.patch('sys.warnoptions', [])\n     def test_run_as_module(self):\n         self.assertEqual(\n             autoreload.get_child_arguments(),\n             [sys.executable, '-m', 'django', 'runserver']\n         )\n \n+    @mock.patch('sys.argv', [__file__, 'runserver'])\n+    @mock.patch('sys.warnoptions', [])\n+    def test_main_module_is_resolved_after_django_main(self):\n+        stale_main = types.ModuleType('__main__')\n+        stale_main.__spec__ = types.SimpleNamespace(parent='stale.package')\n+        active_main = types.ModuleType('__main__')\n+        active_main.__spec__ = types.SimpleNamespace(parent='pkg_other_than_django')\n+        original_import = __import__\n+\n+        def import_module(name, globals=None, locals=None, fromlist=(), level=0):\n+            if name == 'django.__main__':\n+                sys.modules['__main__'] = active_main\n+            return original_import(\n+                name, globals, locals, fromlist, level,\n+            )\n+\n+        with mock.patch.dict(sys.modules, {'__main__': stale_main}):\n+            with mock.patch('builtins.__import__', side_effect=import_module):\n+                self.assertEqual(\n+                    autoreload.get_child_arguments(),\n+                    [sys.executable, '-m', 'pkg_other_than_django', 'runserver'],\n+                )\n+\n     @mock.patch('sys.argv', [__file__, 'runserver'])\n     @mock.patch('sys.warnoptions', ['error'])\n     def test_warnoptions(self):\n",
  "test_command": "python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_main_module_is_resolved_after_django_main"
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
      "duration_seconds": 1.473,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.401,
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
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_main_module_is_resolved_after_django_main (utils_tests.test_autoreload.TestChildArguments)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.6/unittest/mock.py", line 1183, in patched
    return func(*args, **keywargs)
  File "/testbed/tests/utils_tests/test_autoreload.py", line 188, in test_main_module_is_resolved_after_django_main
    [sys.executable, '-m', 'pkg_other_than_django', 'runserver'],
AssertionError: Lists differ: ['/op[19 chars]estbed/bin/python', '-m', 'stale.package', 'runserver'] != ['/op[19 chars]estbed/bin/python', '-m', 'pkg_other_than_django', 'runserver']

First differing element 2:
'stale.package'
'pkg_other_than_django'

- ['/opt/miniconda3/envs/testbed/bin/python', '-m', 'stale.package', 'runserver']
+ ['/opt/miniconda3/envs/testbed/bin/python',
+  '-m',
+  'pkg_other_than_django',
+  'runserver']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -216,14 +216,14 @@ def get_child_arguments():
     executable is reported to not have the .exe extension which can cause bugs
     on reloading.
     """
-    import django.__main__
-    django_main_path = Path(django.__main__.__file__)
+    import __main__
     py_script = Path(sys.argv[0])
 
     args = [sys.executable] + ['-W%s' % o for o in sys.warnoptions]
-    if py_script == django_main_path:
-        # The server was started with `python -m django runserver`.
-        args += ['-m', 'django']
+    # __spec__ is set when the server was started with the `-m` option,
+    # see https://docs.python.org/3/reference/import.html#main-spec
+    if __main__.__spec__ is not None and __main__.__spec__.parent:
+        args += ['-m', __main__.__spec__.parent]
         args += sys.argv[1:]
     elif not py_script.exists():
         # sys.argv[0] may not exist for several reasons on Windows.

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.445,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_main_module_is_resolved_after_django_main"
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
  "reference_log_sha256": "aa90c1bd3a689724595311836effe9b9b283b867062b6b5156413626e8e4ffa2"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_main_module_is_resolved_after_django_main (utils_tests.test_autoreload.TestChildArguments)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.6/unittest/mock.py", line 1183, in patched
    return func(*args, **keywargs)
  File "/testbed/tests/utils_tests/test_autoreload.py", line 188, in test_main_module_is_resolved_after_django_main
    [sys.executable, '-m', 'pkg_other_than_django', 'runserver'],
AssertionError: Lists differ: ['/op[19 chars]estbed/bin/python', '-m', 'stale.package', 'runserver'] != ['/op[19 chars]estbed/bin/python', '-m', 'pkg_other_than_django', 'runserver']

First differing element 2:
'stale.package'
'pkg_other_than_django'

- ['/opt/miniconda3/envs/testbed/bin/python', '-m', 'stale.package', 'runserver']
+ ['/opt/miniconda3/envs/testbed/bin/python',
+  '-m',
+  'pkg_other_than_django',
+  'runserver']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

</gold_execution_log>
