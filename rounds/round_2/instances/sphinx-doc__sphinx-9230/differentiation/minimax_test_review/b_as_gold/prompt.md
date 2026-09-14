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
diff --git a/sphinx/util/docfields.py b/sphinx/util/docfields.py
index 3a3367ebe..6d48e910c 100644
--- a/sphinx/util/docfields.py
+++ b/sphinx/util/docfields.py
@@ -298,7 +298,7 @@ class DocFieldTransformer:
             # also support syntax like ``:param type name:``
             if typedesc.is_typed:
                 try:
-                    argtype, argname = fieldarg.split(None, 1)
+                    argtype, argname = fieldarg.rsplit(None, 1)
                 except ValueError:
                     pass
                 else:

</candidate_a_patch>
<candidate_b_patch>
diff --git a/sphinx/util/docfields.py b/sphinx/util/docfields.py
index 3a3367ebe..214b6e2a7 100644
--- a/sphinx/util/docfields.py
+++ b/sphinx/util/docfields.py
@@ -18,6 +18,43 @@ from sphinx import addnodes
 from sphinx.environment import BuildEnvironment
 from sphinx.util.typing import TextlikeNode
 
+
+
+def _split_type_and_name(fieldarg: str) -> Tuple[str, str]:
+    """Split a field argument like "Dict[str, str] name" into type and name.
+    
+    This handles cases where the type contains brackets or parentheses with
+    spaces inside, like dict(str, str) or List[Tuple[int, str]].
+    Also handles quoted strings like Literal["["].
+    """
+    # Find the last whitespace that is not inside parentheses, brackets, or quotes
+    depth = 0
+    in_string = False
+    string_char = None
+    last_space = -1
+    for i, char in enumerate(fieldarg):
+        # Handle string literals (both single and double quotes)
+        if char in ('"', "'") and not in_string:
+            in_string = True
+            string_char = char
+        elif char == string_char and in_string:
+            in_string = False
+            string_char = None
+        # Handle brackets only when not in a string
+        elif not in_string:
+            if char in '([':
+                depth += 1
+            elif char in ')]':
+                depth -= 1
+            elif char.isspace() and depth == 0:
+                last_space = i
+    
+    if last_space > 0:
+        return fieldarg[:last_space], fieldarg[last_space + 1:].lstrip()
+    else:
+        # No space found, treat the whole thing as the name
+        return '', fieldarg
+
 if TYPE_CHECKING:
     from sphinx.directive import ObjectDescription
 
@@ -298,7 +335,7 @@ class DocFieldTransformer:
             # also support syntax like ``:param type name:``
             if typedesc.is_typed:
                 try:
-                    argtype, argname = fieldarg.split(None, 1)
+                    argtype, argname = _split_type_and_name(fieldarg)
                 except ValueError:
                     pass
                 else:

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_info_field_list_nested_whitespace_is_not_separator",
  "specification_gap": "A :param argument uses inline TYPE NAME syntax only when separated by top-level whitespace. Whitespace inside parentheses is part of the argument; without a top-level separator, the complete argument remains the parameter name and must match a separate :type field.",
  "input_description": "Parse a Python function directive with `:param dict(str, str): optional metadata` and the separate declaration `:type dict(str, str): Mapping`.",
  "expected_output": "The rendered Parameters field body is exactly `dict(str, str) (Mapping) -- optional metadata`. Candidate A instead splits at the whitespace inside the parentheses and rearranges the name and type.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "It checks the complete user-visible field text rather than internal node chunking. The separate :type declaration also makes the intended interpretation unambiguous and avoids relying on an empty inferred type.",
  "test_patch": "diff --git a/tests/test_domain_py.py b/tests/test_domain_py.py\n--- a/tests/test_domain_py.py\n+++ b/tests/test_domain_py.py\n@@ -983,6 +983,19 @@ def test_info_field_list(app):\n                 refdomain=\"py\", reftype=\"class\", reftarget=\"str\",\n                 **{\"py:module\": \"example\", \"py:class\": \"Class\"})\n \n \n+def test_info_field_list_nested_whitespace_is_not_separator(app):\n+    text = (\".. py:function:: example()\\n\"\n+            \"\\n\"\n+            \"   :param dict(str, str): optional metadata\\n\"\n+            \"   :type dict(str, str): Mapping\\n\")\n+    doctree = restructuredtext.parse(app, text)\n+\n+    parameter_field = list(doctree.traverse(nodes.field))[0]\n+    assert parameter_field[1].astext() == (\n+        \"dict(str, str) (Mapping) -- optional metadata\"\n+    )\n+\n+\n def test_info_field_list_var(app):\n     text = (\".. py:class:: Class\\n\"\n",
  "test_command": "cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_nested_whitespace_is_not_separator"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.484,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.374,
      "log_path": "02_execution/attempt_02/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
___________ test_info_field_list_nested_whitespace_is_not_separator ____________

app = <SphinxTestApp buildername='html'>

    def test_info_field_list_nested_whitespace_is_not_separator(app):
        text = (".. py:function:: example()\n"
                "\n"
                "   :param dict(str, str): optional metadata\n"
                "   :type dict(str, str): Mapping\n")
        doctree = restructuredtext.parse(app, text)
    
        parameter_field = list(doctree.traverse(nodes.field))[0]
>       assert parameter_field[1].astext() == (
            "dict(str, str) (Mapping) -- optional metadata"
        )
E       AssertionError: assert 'str) (dict(s...onal metadata' == 'dict(str, st...onal metadata'
E         
E         - dict(str, str) (Mapping) -- optional metadata
E         ?          ---- ----------
E         + str) (dict(str,) -- optional metadata
E         ? ++++++

tests/test_domain_py.py:995: AssertionError
--------------------------- Captured stdout teardown ---------------------------
# testroot: root
# builder: html
# srcdir: /tmp/pytest-of-root/pytest-0/root
# outdir: /tmp/pytest-of-root/pytest-0/root/_build/html
# status: 
[01mRunning Sphinx v4.1.0[39;49;00m

# warning: 

=============================== warnings summary ===============================
sphinx/util/docutils.py:44
  /testbed/sphinx/util/docutils.py:44: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    __version_info__ = tuple(LooseVersion(docutils.__version__).version)

sphinx/highlighting.py:67
  /testbed/sphinx/highlighting.py:67: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if tuple(LooseVersion(pygmentsversion).version) <= (2, 7, 4):

sphinx/registry.py:24
  /testbed/sphinx/registry.py:24: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    from pkg_resources import iter_entry_points

../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('sphinxcontrib')`.
  Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    declare_namespace(pkg)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED tests/test_domain_py.py::test_info_field_list_nested_whitespace_is_not_separator
1 failed, 7 warnings in 0.46s
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
/inputs/candidate.patch:13: trailing whitespace.
    
/inputs/candidate.patch:39: trailing whitespace.
    
warning: 2 lines add whitespace errors.
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
=============================== warnings summary ===============================
sphinx/util/docutils.py:44
  /testbed/sphinx/util/docutils.py:44: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    __version_info__ = tuple(LooseVersion(docutils.__version__).version)

sphinx/highlighting.py:67
  /testbed/sphinx/highlighting.py:67: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if tuple(LooseVersion(pygmentsversion).version) <= (2, 7, 4):

sphinx/registry.py:24
  /testbed/sphinx/registry.py:24: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    from pkg_resources import iter_entry_points

../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('sphinxcontrib')`.
  Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    declare_namespace(pkg)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
1 passed, 7 warnings in 0.39s
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/sphinx/util/docfields.py b/sphinx/util/docfields.py
--- a/sphinx/util/docfields.py
+++ b/sphinx/util/docfields.py
@@ -298,7 +298,7 @@ def transform(self, node: nodes.field_list) -> None:
             # also support syntax like ``:param type name:``
             if typedesc.is_typed:
                 try:
-                    argtype, argname = fieldarg.split(None, 1)
+                    argtype, argname = fieldarg.rsplit(None, 1)
                 except ValueError:
                     pass
                 else:

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.547,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_info_field_list_nested_whitespace_is_not_separator"
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
  "reference_log_sha256": "d1a0965d7e22c8b136bd1d42e13aa45dd9d17c632fd1566b2a9013380b47cbaa"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
___________ test_info_field_list_nested_whitespace_is_not_separator ____________

app = <SphinxTestApp buildername='html'>

    def test_info_field_list_nested_whitespace_is_not_separator(app):
        text = (".. py:function:: example()\n"
                "\n"
                "   :param dict(str, str): optional metadata\n"
                "   :type dict(str, str): Mapping\n")
        doctree = restructuredtext.parse(app, text)
    
        parameter_field = list(doctree.traverse(nodes.field))[0]
>       assert parameter_field[1].astext() == (
            "dict(str, str) (Mapping) -- optional metadata"
        )
E       AssertionError: assert 'str) (dict(s...onal metadata' == 'dict(str, st...onal metadata'
E         
E         - dict(str, str) (Mapping) -- optional metadata
E         ?          ---- ----------
E         + str) (dict(str,) -- optional metadata
E         ? ++++++

tests/test_domain_py.py:995: AssertionError
--------------------------- Captured stdout teardown ---------------------------
# testroot: root
# builder: html
# srcdir: /tmp/pytest-of-root/pytest-0/root
# outdir: /tmp/pytest-of-root/pytest-0/root/_build/html
# status: 
[01mRunning Sphinx v4.1.0[39;49;00m

# warning: 

=============================== warnings summary ===============================
sphinx/util/docutils.py:44
  /testbed/sphinx/util/docutils.py:44: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    __version_info__ = tuple(LooseVersion(docutils.__version__).version)

sphinx/highlighting.py:67
  /testbed/sphinx/highlighting.py:67: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if tuple(LooseVersion(pygmentsversion).version) <= (2, 7, 4):

sphinx/registry.py:24
  /testbed/sphinx/registry.py:24: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    from pkg_resources import iter_entry_points

../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
../opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154
  /opt/miniconda3/envs/testbed/lib/python3.9/site-packages/pkg_resources/__init__.py:3154: DeprecationWarning: Deprecated call to `pkg_resources.declare_namespace('sphinxcontrib')`.
  Implementing implicit namespace packages (as specified in PEP 420) is preferred to `pkg_resources.declare_namespace`. See https://setuptools.pypa.io/en/latest/references/keywords.html#keyword-namespace-packages
    declare_namespace(pkg)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED tests/test_domain_py.py::test_info_field_list_nested_whitespace_is_not_separator
1 failed, 7 warnings in 0.47s
[pipeline] test_exit_code=1

</gold_execution_log>
