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
Allow autoreloading of `python -m pkg_other_than_django runserver`
Description
	 
		(last modified by William Schwartz)
	 
​django.utils.autoreload.get_child_arguments detects if Python was launched as python -m django. Currently it detects only when ​-m was passed specifically django (and only in Python environments in which __file__ is set on modules, which is ​not true of all Python environments). Like #32177, this ticket aims to remove one impediment to creating Django-based command-line utilities that have their own ​__main__ sub-module while overriding Django's built-in management commands—in this case, runserver.
The fix, which I have submitted in the ​attached PR, is to use Python's ​documented way of determining if -m was used in get_child_arguments:
The top-level __main__ module is always the entry point of a ​complete Python program.
 __main__.__spec__ is not None ​if and only if Python was launched with -m or the name of a "directory, zipfile or other sys.path entry." In the latter cases, the ​documentation says
If the script name refers to a directory or zipfile, the script name is added to the start of sys.path and the __main__.py file in that location is executed as the __main__ module.
Hence __main__.__spec__.parent (which is ​usually but not always __main__.__package__) exists and is the empty string when Python is started with the name of a directory or zip file.
Therefore Python was started with -m pkg if and only if __main__.__spec__.parent == "pkg".
Following this algorithm is guaranteed to work as long as Python obeys its own documentation, and has the side benefit of avoiding use of __file__.

</issue_statement>
<candidate_a_patch>
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
index b8efb9f881..e0a8484aee 100644
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -216,14 +216,15 @@ def get_child_arguments():
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
+    # Check if Python was launched with -m pkg
+    # __main__.__spec__ is set when using -m, and __spec__.parent gives the package name
+    if getattr(__main__.__spec__, 'parent', None):
+        # The server was started with `python -m pkg runserver`.
+        args += ['-m', __main__.__spec__.parent]
         args += sys.argv[1:]
     elif not py_script.exists():
         # sys.argv[0] may not exist for several reasons on Windows.

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
index b8efb9f881..01e10e8df6 100644
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -221,7 +221,17 @@ def get_child_arguments():
     py_script = Path(sys.argv[0])
 
     args = [sys.executable] + ['-W%s' % o for o in sys.warnoptions]
-    if py_script == django_main_path:
+    # Check if Python was started with -m option
+    # __main__.__spec__ is set when Python is launched with -m
+    # __spec__.parent is the package name (e.g., 'django' for 'python -m django')
+    # It is non-None and non-empty only when -m is used with a package
+    # Use sys.modules to access __main__ at runtime to handle mocking in tests
+    main_module = sys.modules['__main__']
+    if getattr(main_module.__spec__, 'parent', None):
+        # The server was started with `python -m package runserver`.
+        args += ['-m', main_module.__spec__.parent]
+        args += sys.argv[1:]
+    elif py_script == django_main_path:
         # The server was started with `python -m django runserver`.
         args += ['-m', 'django']
         args += sys.argv[1:]

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_run_as_django_module_without_main_spec",
  "specification_gap": "Support for arbitrary `python -m package` entry points must not remove the existing fallback for `python -m django` when `__main__.__spec__` is unavailable. In that case, Django's known `__main__.py` path still identifies the module invocation.",
  "input_description": "Set `sys.argv` to `[django.__main__.__file__, 'runserver']`, clear warning options, set the running `__main__.__spec__` to `None`, and call `autoreload.get_child_arguments()`.",
  "expected_output": "The returned child argument list is `[sys.executable, '-m', 'django', 'runserver']`, preserving module execution instead of restarting Django's `__main__.py` as an ordinary script.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the metadata-unavailable boundary of the established Django module entry point. Candidate_b retains the path-based compatibility fallback, while candidate_a removes it and returns `[sys.executable, django.__main__.__file__, 'runserver']`.",
  "test_patch": "diff --git a/tests/utils_tests/test_autoreload.py b/tests/utils_tests/test_autoreload.py\n--- a/tests/utils_tests/test_autoreload.py\n+++ b/tests/utils_tests/test_autoreload.py\n@@ -160,9 +160,18 @@ class TestChildArguments(SimpleTestCase):\n     @mock.patch('sys.argv', [django.__main__.__file__, 'runserver'])\n     @mock.patch('sys.warnoptions', [])\n     def test_run_as_module(self):\n         self.assertEqual(\n             autoreload.get_child_arguments(),\n             [sys.executable, '-m', 'django', 'runserver']\n         )\n \n+    @mock.patch.object(sys.modules['__main__'], '__spec__', None)\n+    @mock.patch('sys.argv', [django.__main__.__file__, 'runserver'])\n+    @mock.patch('sys.warnoptions', [])\n+    def test_run_as_django_module_without_main_spec(self):\n+        self.assertEqual(\n+            autoreload.get_child_arguments(),\n+            [sys.executable, '-m', 'django', 'runserver']\n+        )\n+\n     @mock.patch('sys.argv', [__file__, 'runserver'])\n",
  "test_command": "cd /testbed && ./tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_run_as_django_module_without_main_spec"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.43,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.348,
      "log_path": "02_execution/attempt_01/candidate_b.log"
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
F
======================================================================
FAIL: test_run_as_django_module_without_main_spec (utils_tests.test_autoreload.TestChildArguments)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.6/unittest/mock.py", line 1183, in patched
    return func(*args, **keywargs)
  File "/testbed/tests/utils_tests/test_autoreload.py", line 174, in test_run_as_django_module_without_main_spec
    [sys.executable, '-m', 'django', 'runserver']
AssertionError: Lists differ: ['/op[19 chars]estbed/bin/python', '/testbed/django/__main__.py', 'runserver'] != ['/op[19 chars]estbed/bin/python', '-m', 'django', 'runserver']

First differing element 1:
'/testbed/django/__main__.py'
'-m'

Second list contains 1 additional elements.
First extra element 3:
'runserver'

+ ['/opt/miniconda3/envs/testbed/bin/python', '-m', 'django', 'runserver']
- ['/opt/miniconda3/envs/testbed/bin/python',
-  '/testbed/django/__main__.py',
-  'runserver']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

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
  "duration_seconds": 1.402,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_run_as_django_module_without_main_spec"
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
  "reference_log_sha256": "4fd49d79c116e2daa26cf3e1f04484870b47852bff76eab576695f5a96c48da2"
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
FAIL: test_run_as_django_module_without_main_spec (utils_tests.test_autoreload.TestChildArguments)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.6/unittest/mock.py", line 1183, in patched
    return func(*args, **keywargs)
  File "/testbed/tests/utils_tests/test_autoreload.py", line 174, in test_run_as_django_module_without_main_spec
    [sys.executable, '-m', 'django', 'runserver']
AssertionError: Lists differ: ['/op[19 chars]estbed/bin/python', '/testbed/django/__main__.py', 'runserver'] != ['/op[19 chars]estbed/bin/python', '-m', 'django', 'runserver']

First differing element 1:
'/testbed/django/__main__.py'
'-m'

Second list contains 1 additional elements.
First extra element 3:
'runserver'

+ ['/opt/miniconda3/envs/testbed/bin/python', '-m', 'django', 'runserver']
- ['/opt/miniconda3/envs/testbed/bin/python',
-  '/testbed/django/__main__.py',
-  'runserver']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

</gold_execution_log>
