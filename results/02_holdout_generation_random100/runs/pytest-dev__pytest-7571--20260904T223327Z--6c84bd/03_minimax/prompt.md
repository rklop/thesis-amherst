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
caplog fixture doesn't restore log level after test
From the documentation at https://docs.pytest.org/en/6.0.0/logging.html#caplog-fixture, "The log levels set are restored automatically at the end of the test".
It used to work, but looks broken in new 6.0 release. Minimal example to reproduce:

```
def test_foo(caplog):
    caplog.set_level(42)

def test_bar(caplog):
    print(caplog.handler.level)
```

It prints "0" for pytest<6, "42" after.

</issue_statement>

<original_test_patch>
diff --git a/testing/logging/test_fixture.py b/testing/logging/test_fixture.py
--- a/testing/logging/test_fixture.py
+++ b/testing/logging/test_fixture.py
@@ -2,6 +2,7 @@
 
 import pytest
 from _pytest.logging import caplog_records_key
+from _pytest.pytester import Testdir
 
 logger = logging.getLogger(__name__)
 sublogger = logging.getLogger(__name__ + ".baz")
@@ -27,8 +28,11 @@ def test_change_level(caplog):
     assert "CRITICAL" in caplog.text
 
 
-def test_change_level_undo(testdir):
-    """Ensure that 'set_level' is undone after the end of the test"""
+def test_change_level_undo(testdir: Testdir) -> None:
+    """Ensure that 'set_level' is undone after the end of the test.
+
+    Tests the logging output themselves (affacted both by logger and handler levels).
+    """
     testdir.makepyfile(
         """
         import logging
@@ -50,6 +54,33 @@ def test2(caplog):
     result.stdout.no_fnmatch_line("*log from test2*")
 
 
+def test_change_level_undos_handler_level(testdir: Testdir) -> None:
+    """Ensure that 'set_level' is undone after the end of the test (handler).
+
+    Issue #7569. Tests the handler level specifically.
+    """
+    testdir.makepyfile(
+        """
+        import logging
+
+        def test1(caplog):
+            assert caplog.handler.level == 0
+            caplog.set_level(41)
+            assert caplog.handler.level == 41
+
+        def test2(caplog):
+            assert caplog.handler.level == 0
+
+        def test3(caplog):
+            assert caplog.handler.level == 0
+            caplog.set_level(43)
+            assert caplog.handler.level == 43
+    """
+    )
+    result = testdir.runpytest()
+    result.assert_outcomes(passed=3)
+
+
 def test_with_statement(caplog):
     with caplog.at_level(logging.INFO):
         logger.debug("handler DEBUG level")

</original_test_patch>

<candidate_a_patch>
diff --git a/src/_pytest/logging.py b/src/_pytest/logging.py
index 11031f2f2..1b4d9d385 100644
--- a/src/_pytest/logging.py
+++ b/src/_pytest/logging.py
@@ -346,6 +346,7 @@ class LogCaptureFixture:
         self._item = item
         # dict of log name -> log level
         self._initial_logger_levels = {}  # type: Dict[Optional[str], int]
+        self._initial_handler_level = None  # type: Optional[int]
 
     def _finalize(self) -> None:
         """Finalizes the fixture.
@@ -356,6 +357,9 @@ class LogCaptureFixture:
         for logger_name, level in self._initial_logger_levels.items():
             logger = logging.getLogger(logger_name)
             logger.setLevel(level)
+        # restore handler level
+        if self._initial_handler_level is not None:
+            self.handler.setLevel(self._initial_handler_level)
 
     @property
     def handler(self) -> LogCaptureHandler:
@@ -434,6 +438,9 @@ class LogCaptureFixture:
         # save the original log-level to restore it during teardown
         self._initial_logger_levels.setdefault(logger, logger_obj.level)
         logger_obj.setLevel(level)
+        # save the original handler level to restore it during teardown
+        if self._initial_handler_level is None:
+            self._initial_handler_level = self.handler.level
         self.handler.setLevel(level)
 
     @contextmanager

</candidate_a_patch>

<candidate_b_patch>
diff --git a/src/_pytest/logging.py b/src/_pytest/logging.py
--- a/src/_pytest/logging.py
+++ b/src/_pytest/logging.py
@@ -345,6 +345,7 @@ def __init__(self, item: nodes.Node) -> None:
         """Creates a new funcarg."""
         self._item = item
         # dict of log name -> log level
+        self._initial_handler_level = None  # type: Optional[int]
         self._initial_logger_levels = {}  # type: Dict[Optional[str], int]
 
     def _finalize(self) -> None:
@@ -353,6 +354,8 @@ def _finalize(self) -> None:
         This restores the log levels changed by :meth:`set_level`.
         """
         # restore log levels
+        if self._initial_handler_level is not None:
+            self.handler.setLevel(self._initial_handler_level)
         for logger_name, level in self._initial_logger_levels.items():
             logger = logging.getLogger(logger_name)
             logger.setLevel(level)
@@ -434,6 +437,7 @@ def set_level(self, level: Union[int, str], logger: Optional[str] = None) -> Non
         # save the original log-level to restore it during teardown
         self._initial_logger_levels.setdefault(logger, logger_obj.level)
         logger_obj.setLevel(level)
+        self._initial_handler_level = self.handler.level
         self.handler.setLevel(level)
 
     @contextmanager

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_set_level_after_at_level_restores_effective_level",
  "specification_gap": "When caplog.set_level() is used inside caplog.at_level(), the context manager restores the handler when it exits. A subsequent set_level() call must preserve that newly effective pre-call level for test teardown, rather than retaining the temporary at_level() threshold.",
  "input_description": "The first nested test calls set_level(INFO) inside an at_level(ERROR) context, exits the context, then calls set_level(DEBUG). A following test emits a WARNING record through a normally configured logger.",
  "expected_output": "The temporary ERROR handler threshold does not leak into the following test, so the WARNING record appears in caplog.messages and the nested run reports two passing tests.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests composition of two public caplog APIs and observes actual capture behavior across tests. Candidate B updates the saved handler level after at_level() restores it; candidate A retains the transient ERROR level and filters the following WARNING record.",
  "test_patch": "diff --git a/testing/logging/test_caplog_restore_interaction.py b/testing/logging/test_caplog_restore_interaction.py\nnew file mode 100644\nindex 000000000..e413a0d01\n--- /dev/null\n+++ b/testing/logging/test_caplog_restore_interaction.py\n@@ -0,0 +1,23 @@\n+def test_set_level_after_at_level_restores_effective_level(testdir):\n+    testdir.makepyfile(\n+        \"\"\"\n+        import logging\n+\n+        def test_configure(caplog):\n+            with caplog.at_level(logging.ERROR, logger=\"temporary\"):\n+                caplog.set_level(logging.INFO, logger=\"inside\")\n+            caplog.set_level(logging.DEBUG, logger=\"outside\")\n+\n+        def test_handler_was_restored(caplog):\n+            logger = logging.getLogger(\"probe\")\n+            old_level = logger.level\n+            logger.setLevel(logging.WARNING)\n+            try:\n+                logger.warning(\"still captured\")\n+            finally:\n+                logger.setLevel(old_level)\n+            assert \"still captured\" in caplog.messages\n+        \"\"\"\n+    )\n+    result = testdir.runpytest(\"-q\")\n+    result.assert_outcomes(passed=2)\n",
  "test_command": "pytest -q testing/logging/test_caplog_restore_interaction.py::test_set_level_after_at_level_restores_effective_level"
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
F                                                                        [100%]
=================================== FAILURES ===================================
____________ test_set_level_after_at_level_restores_effective_level ____________

testdir = <Testdir local('/tmp/pytest-of-root/pytest-0/test_set_level_after_at_level_restores_effective_level0')>

    def test_set_level_after_at_level_restores_effective_level(testdir):
        testdir.makepyfile(
            """
            import logging
    
            def test_configure(caplog):
                with caplog.at_level(logging.ERROR, logger="temporary"):
                    caplog.set_level(logging.INFO, logger="inside")
                caplog.set_level(logging.DEBUG, logger="outside")
    
            def test_handler_was_restored(caplog):
                logger = logging.getLogger("probe")
                old_level = logger.level
                logger.setLevel(logging.WARNING)
                try:
                    logger.warning("still captured")
                finally:
                    logger.setLevel(old_level)
                assert "still captured" in caplog.messages
            """
        )
        result = testdir.runpytest("-q")
>       result.assert_outcomes(passed=2)
E       AssertionError: assert {'errors': 0,...pped': 0, ...} == {'errors': 0,...pped': 0, ...}
E         Omitting 4 identical items, use -vv to show
E         Differing items:
E         {'passed': 1} != {'passed': 2}
E         {'failed': 1} != {'failed': 0}
E         Full diff:
E           {
E            'errors': 0,...
E         
E         ...Full output truncated (13 lines hidden), use '-vv' to show

/testbed/testing/logging/test_caplog_restore_interaction.py:23: AssertionError
----------------------------- Captured stdout call -----------------------------
.F                                                                       [100%]
=================================== FAILURES ===================================
__________________________ test_handler_was_restored ___________________________

caplog = <_pytest.logging.LogCaptureFixture object at 0x75c60f6c8820>

    def test_handler_was_restored(caplog):
        logger = logging.getLogger("probe")
        old_level = logger.level
        logger.setLevel(logging.WARNING)
        try:
            logger.warning("still captured")
        finally:
            logger.setLevel(old_level)
>       assert "still captured" in caplog.messages
E       AssertionError: assert 'still captured' in []
E        +  where [] = <_pytest.logging.LogCaptureFixture object at 0x75c60f6c8820>.messages

test_set_level_after_at_level_restores_effective_level.py:16: AssertionError
------------------------------ Captured log call -------------------------------
WARNING  probe:test_set_level_after_at_level_restores_effective_level.py:13 still captured
=========================== short test summary info ============================
FAILED test_set_level_after_at_level_restores_effective_level.py::test_handler_was_restored
1 failed, 1 passed in 0.02s
------------------------------ Captured log call -------------------------------
WARNING  probe:test_set_level_after_at_level_restores_effective_level.py:13 still captured
=========================== short test summary info ============================
FAILED testing/logging/test_caplog_restore_interaction.py::test_set_level_after_at_level_restores_effective_level
1 failed in 0.03s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed in 0.02s
[pipeline] test_exit_code=0

```
</validated_execution>
