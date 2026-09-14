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
docutils reports an error rendering view docstring when the first line is not empty
Description
	
Currently admindoc works correctly only with docstrings where the first line is empty, and all Django docstrings are formatted in this way.
However usually the docstring text starts at the first line, e.g.:
def test():
	"""test tests something.
	"""
and this cause an error:
Error in "default-role" directive:
no content permitted.
.. default-role:: cmsreference
The culprit is this code in trim_docstring:
indent = min(len(line) - len(line.lstrip()) for line in lines if line.lstrip())
The problem is that the indentation of the first line is 0.
The solution is to skip the first line:
indent = min(len(line) - len(line.lstrip()) for line in lines[1:] if line.lstrip())
Thanks.

</issue_statement>

<original_test_patch>
diff --git a/tests/admin_docs/test_utils.py b/tests/admin_docs/test_utils.py
--- a/tests/admin_docs/test_utils.py
+++ b/tests/admin_docs/test_utils.py
@@ -1,8 +1,9 @@
 import unittest
 
 from django.contrib.admindocs.utils import (
-    docutils_is_available, parse_docstring, parse_rst, trim_docstring,
+    docutils_is_available, parse_docstring, parse_rst,
 )
+from django.test.utils import captured_stderr
 
 from .tests import AdminDocsSimpleTestCase
 
@@ -31,19 +32,6 @@ class TestUtils(AdminDocsSimpleTestCase):
     def setUp(self):
         self.docstring = self.__doc__
 
-    def test_trim_docstring(self):
-        trim_docstring_output = trim_docstring(self.docstring)
-        trimmed_docstring = (
-            'This __doc__ output is required for testing. I copied this '
-            'example from\n`admindocs` documentation. (TITLE)\n\n'
-            'Display an individual :model:`myapp.MyModel`.\n\n'
-            '**Context**\n\n``RequestContext``\n\n``mymodel``\n'
-            '    An instance of :model:`myapp.MyModel`.\n\n'
-            '**Template:**\n\n:template:`myapp/my_template.html` '
-            '(DESCRIPTION)\n\nsome_metadata: some data'
-        )
-        self.assertEqual(trim_docstring_output, trimmed_docstring)
-
     def test_parse_docstring(self):
         title, description, metadata = parse_docstring(self.docstring)
         docstring_title = (
@@ -106,6 +94,13 @@ def test_parse_rst(self):
         self.assertEqual(parse_rst('`title`', 'filter'), markup % 'filters/#title')
         self.assertEqual(parse_rst('`title`', 'tag'), markup % 'tags/#title')
 
+    def test_parse_rst_with_docstring_no_leading_line_feed(self):
+        title, body, _ = parse_docstring('firstline\n\n    second line')
+        with captured_stderr() as stderr:
+            self.assertEqual(parse_rst(title, ''), '<p>firstline</p>\n')
+            self.assertEqual(parse_rst(body, ''), '<p>second line</p>\n')
+        self.assertEqual(stderr.getvalue(), '')
+
     def test_publish_parts(self):
         """
         Django shouldn't break the default role for interpreted text

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/admindocs/utils.py b/django/contrib/admindocs/utils.py
index 4c0e7e2a56..aa663ead5b 100644
--- a/django/contrib/admindocs/utils.py
+++ b/django/contrib/admindocs/utils.py
@@ -34,7 +34,7 @@ def trim_docstring(docstring):
         return ''
     # Convert tabs to spaces and split into lines
     lines = docstring.expandtabs().splitlines()
-    indent = min(len(line) - len(line.lstrip()) for line in lines if line.lstrip())
+    indent = min((len(line) - len(line.lstrip()) for line in lines[1:] if line.lstrip()), default=0)
     trimmed = [lines[0].lstrip()] + [line[indent:].rstrip() for line in lines[1:]]
     return "\n".join(trimmed).strip()
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/admindocs/utils.py b/django/contrib/admindocs/utils.py
--- a/django/contrib/admindocs/utils.py
+++ b/django/contrib/admindocs/utils.py
@@ -3,6 +3,7 @@
 import re
 from email.errors import HeaderParseError
 from email.parser import HeaderParser
+from inspect import cleandoc
 
 from django.urls import reverse
 from django.utils.regex_helper import _lazy_re_compile
@@ -24,26 +25,13 @@ def get_view_name(view_func):
     return mod_name + '.' + view_name
 
 
-def trim_docstring(docstring):
-    """
-    Uniformly trim leading/trailing whitespace from docstrings.
-
-    Based on https://www.python.org/dev/peps/pep-0257/#handling-docstring-indentation
-    """
-    if not docstring or not docstring.strip():
-        return ''
-    # Convert tabs to spaces and split into lines
-    lines = docstring.expandtabs().splitlines()
-    indent = min(len(line) - len(line.lstrip()) for line in lines if line.lstrip())
-    trimmed = [lines[0].lstrip()] + [line[indent:].rstrip() for line in lines[1:]]
-    return "\n".join(trimmed).strip()
-
-
 def parse_docstring(docstring):
     """
     Parse out the parts of a docstring.  Return (title, body, metadata).
     """
-    docstring = trim_docstring(docstring)
+    if not docstring:
+        return '', '', {}
+    docstring = cleandoc(docstring)
     parts = re.split(r'\n{2,}', docstring)
     title = parts[0]
     if len(parts) == 1:
diff --git a/django/contrib/admindocs/views.py b/django/contrib/admindocs/views.py
--- a/django/contrib/admindocs/views.py
+++ b/django/contrib/admindocs/views.py
@@ -1,5 +1,6 @@
 import inspect
 from importlib import import_module
+from inspect import cleandoc
 from pathlib import Path
 
 from django.apps import apps
@@ -256,7 +257,7 @@ def get_context_data(self, **kwargs):
                     continue
                 verbose = func.__doc__
                 verbose = verbose and (
-                    utils.parse_rst(utils.trim_docstring(verbose), 'model', _('model:') + opts.model_name)
+                    utils.parse_rst(cleandoc(verbose), 'model', _('model:') + opts.model_name)
                 )
                 # Show properties and methods without arguments as fields.
                 # Otherwise, show as a 'method with arguments'.

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_parse_docstring_preserves_rst_escaped_space",
  "specification_gap": "Docstring indentation cleanup must not strip meaningful interior content. In reStructuredText, a backslash followed by a trailing space is an escape sequence, but candidate A applies rstrip() to every line and removes that space. Candidate B uses inspect.cleandoc(), which removes indentation while preserving it.",
  "input_description": "Pass parse_docstring() the first-line docstring \"Summary.\\n\\n    first\\\\ \\n    second\", whose body contains a reStructuredText escaped space immediately before a newline.",
  "expected_output": "parse_docstring() returns exactly ('Summary.', 'first\\\\ \\nsecond', {}), retaining the space after the backslash while removing the body's four-space indentation.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests content preservation rather than ordinary indentation alone. Removing the escaped space changes reStructuredText syntax, so it exposes a semantic difference between the custom trimming algorithm and the intended general-purpose docstring cleaner.",
  "test_patch": "diff --git a/tests/admin_docs/test_docstring_cleaning.py b/tests/admin_docs/test_docstring_cleaning.py\nnew file mode 100644\n--- /dev/null\n+++ b/tests/admin_docs/test_docstring_cleaning.py\n@@ -0,0 +1,11 @@\n+from django.contrib.admindocs.utils import parse_docstring\n+from django.test import SimpleTestCase\n+\n+\n+class DocstringCleaningTests(SimpleTestCase):\n+\n+    def test_parse_docstring_preserves_rst_escaped_space(self):\n+        docstring = 'Summary.\\n\\n    first\\\\ \\n    second'\n+        self.assertEqual(\n+            parse_docstring(docstring), ('Summary.', 'first\\\\ \\nsecond', {}),\n+        )\n",
  "test_command": "./tests/runtests.py admin_docs.test_docstring_cleaning.DocstringCleaningTests.test_parse_docstring_preserves_rst_escaped_space"
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
F
======================================================================
FAIL: test_parse_docstring_preserves_rst_escaped_space (admin_docs.test_docstring_cleaning.DocstringCleaningTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/admin_docs/test_docstring_cleaning.py", line 10, in test_parse_docstring_preserves_rst_escaped_space
    parse_docstring(docstring), ('Summary.', 'first\\ \nsecond', {}),
AssertionError: Tuples differ: ('Summary.', 'first\\\nsecond', {}) != ('Summary.', 'first\\ \nsecond', {})

First differing element 1:
'first\\\nsecond'
'first\\ \nsecond'

- ('Summary.', 'first\\\nsecond', {})
+ ('Summary.', 'first\\ \nsecond', {})
?                      +


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
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
