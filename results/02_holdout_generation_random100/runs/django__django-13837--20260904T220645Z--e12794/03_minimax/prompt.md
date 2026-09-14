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

<original_test_patch>
diff --git a/tests/utils_tests/test_autoreload.py b/tests/utils_tests/test_autoreload.py
--- a/tests/utils_tests/test_autoreload.py
+++ b/tests/utils_tests/test_autoreload.py
@@ -23,6 +23,7 @@
 from django.utils import autoreload
 from django.utils.autoreload import WatchmanUnavailable
 
+from .test_module import __main__ as test_main
 from .utils import on_macos_with_hfs
 
 
@@ -157,6 +158,7 @@ def test_path_with_embedded_null_bytes(self):
 
 
 class TestChildArguments(SimpleTestCase):
+    @mock.patch.dict(sys.modules, {'__main__': django.__main__})
     @mock.patch('sys.argv', [django.__main__.__file__, 'runserver'])
     @mock.patch('sys.warnoptions', [])
     def test_run_as_module(self):
@@ -165,6 +167,15 @@ def test_run_as_module(self):
             [sys.executable, '-m', 'django', 'runserver']
         )
 
+    @mock.patch.dict(sys.modules, {'__main__': test_main})
+    @mock.patch('sys.argv', [test_main.__file__, 'runserver'])
+    @mock.patch('sys.warnoptions', [])
+    def test_run_as_non_django_module(self):
+        self.assertEqual(
+            autoreload.get_child_arguments(),
+            [sys.executable, '-m', 'utils_tests.test_module', 'runserver'],
+        )
+
     @mock.patch('sys.argv', [__file__, 'runserver'])
     @mock.patch('sys.warnoptions', ['error'])
     def test_warnoptions(self):
@@ -447,7 +458,8 @@ def test_python_m_django(self):
         argv = [main, 'runserver']
         mock_call = self.patch_autoreload(argv)
         with mock.patch('django.__main__.__file__', main):
-            autoreload.restart_with_reloader()
+            with mock.patch.dict(sys.modules, {'__main__': django.__main__}):
+                autoreload.restart_with_reloader()
             self.assertEqual(mock_call.call_count, 1)
             self.assertEqual(mock_call.call_args[0][0], [self.executable, '-Wall', '-m', 'django'] + argv[1:])
 
diff --git a/tests/utils_tests/test_module/__main__.py b/tests/utils_tests/test_module/__main__.py
new file mode 100644

</original_test_patch>

<candidate_a_patch>
diff --git a/django/utils/autoreload.py b/django/utils/autoreload.py
index b8efb9f881..648f42cb23 100644
--- a/django/utils/autoreload.py
+++ b/django/utils/autoreload.py
@@ -216,16 +216,25 @@ def get_child_arguments():
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
-        args += sys.argv[1:]
-    elif not py_script.exists():
+    # __main__.__spec__ is set when the server was started with the `-m` option,
+    # see https://docs.python.org/3/reference/import.html#main-spec
+    # __spec__.parent is the parent package, e.g. 'django' for `python -m django runserver`
+    # or 'utils_tests.test_module' for `python -m utils_tests.test_module runserver`
+    # Additionally, check that sys.argv[0] matches either __main__.__file__ or
+    # __spec__.origin to ensure we're not using a stale __spec__ from a previous
+    # invocation (e.g., in tests where __main__ is mocked).
+    if getattr(__main__, '__spec__', None) is not None and __main__.__spec__.parent:
+        # Check __file__ first (this is what the test mocks), then fall back to __spec__.origin
+        main_path = Path(getattr(__main__, '__file__', '') or __main__.__spec__.origin or '')
+        if main_path.resolve() == py_script.resolve():
+            args += ['-m', __main__.__spec__.parent]
+            args += sys.argv[1:]
+            return args
+    if not py_script.exists():
         # sys.argv[0] may not exist for several reasons on Windows.
         # It may exist with a .exe extension or have a -script.py suffix.
         exe_entrypoint = py_script.with_suffix('.exe')
@@ -241,8 +250,7 @@ def get_child_arguments():
             # args parameter accepts path-like on Windows from Python 3.8.
             return [*args, str(script_entrypoint), *sys.argv[1:]]
         raise RuntimeError('Script %s does not exist.' % py_script)
-    else:
-        args += sys.argv
+    args += sys.argv
     return args
 
 

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_run_as_module_without_main_file",
  "specification_gap": "A nonempty __main__.__spec__.parent is the documented signal that Python used -m, even when __main__.__file__ and __spec__.origin are unavailable. Candidate A incorrectly requires an additional path match; candidate B does not.",
  "input_description": "Set __main__.__spec__.parent to 'custom_cli', set both __main__.__file__ and the spec origin to None, use an existing script path as sys.argv[0], and request runserver.",
  "expected_output": "get_child_arguments() returns [sys.executable, '-m', 'custom_cli', 'runserver'], preserving the module-based public entry point.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This directly covers the issue's stated alternate Python environments where __file__ isn't set and verifies the general -m relaunch invariant for packages other than django.",
  "test_patch": "diff --git a/tests/utils_tests/test_autoreload.py b/tests/utils_tests/test_autoreload.py\n--- a/tests/utils_tests/test_autoreload.py\n+++ b/tests/utils_tests/test_autoreload.py\n@@ -163,6 +163,18 @@ class TestChildArguments(SimpleTestCase):\n         self.assertEqual(\n             autoreload.get_child_arguments(),\n             [sys.executable, '-m', 'django', 'runserver']\n         )\n \n+    @mock.patch('sys.argv', [__file__, 'runserver'])\n+    @mock.patch('sys.warnoptions', [])\n+    def test_run_as_module_without_main_file(self):\n+        main_module = sys.modules['__main__']\n+        module_spec = types.SimpleNamespace(parent='custom_cli', origin=None)\n+        with mock.patch.object(main_module, '__spec__', module_spec):\n+            with mock.patch.object(main_module, '__file__', None):\n+                self.assertEqual(\n+                    autoreload.get_child_arguments(),\n+                    [sys.executable, '-m', 'custom_cli', 'runserver']\n+                )\n+\n     @mock.patch('sys.argv', [__file__, 'runserver'])\n     @mock.patch('sys.warnoptions', ['error'])\n     def test_warnoptions(self):\n",
  "test_command": "python tests/runtests.py utils_tests.test_autoreload.TestChildArguments.test_run_as_module_without_main_file"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_run_as_module_without_main_file (utils_tests.test_autoreload.TestChildArguments)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/opt/miniconda3/envs/testbed/lib/python3.6/unittest/mock.py", line 1183, in patched
    return func(*args, **keywargs)
  File "/testbed/tests/utils_tests/test_autoreload.py", line 177, in test_run_as_module_without_main_file
    [sys.executable, '-m', 'custom_cli', 'runserver']
AssertionError: Lists differ: ['/op[35 chars]n', '/testbed/tests/utils_tests/test_autoreloa[14 chars]ver'] != ['/op[35 chars]n', '-m', 'custom_cli', 'runserver']

First differing element 1:
'/testbed/tests/utils_tests/test_autoreload.py'
'-m'

Second list contains 1 additional elements.
First extra element 3:
'runserver'

+ ['/opt/miniconda3/envs/testbed/bin/python', '-m', 'custom_cli', 'runserver']
- ['/opt/miniconda3/envs/testbed/bin/python',
-  '/testbed/tests/utils_tests/test_autoreload.py',
-  'runserver']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
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

```
</validated_execution>
