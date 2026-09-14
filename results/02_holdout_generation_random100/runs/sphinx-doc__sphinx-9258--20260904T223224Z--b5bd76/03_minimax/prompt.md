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
[RFE] Support union types specification using | (vertical bar/pipe)
Please add a support for specifying multiple types acceptable for a parameter/attribute/variable.
Use case:
Imagine that there is a function that accepts both `bytes` and `str`. The docstring would look like:

``` restructuredtext
def foo(text):
    """Bar

    :param text: a text
    :type text: bytes | str

    """
```

Such a syntax is already supported by e.g. [PyCharm](https://www.jetbrains.com/pycharm/help/type-hinting-in-pycharm.html).


</issue_statement>

<original_test_patch>
diff --git a/tests/test_domain_py.py b/tests/test_domain_py.py
--- a/tests/test_domain_py.py
+++ b/tests/test_domain_py.py
@@ -1009,6 +1009,40 @@ def test_info_field_list(app):
                 **{"py:module": "example", "py:class": "Class"})
 
 
+def test_info_field_list_piped_type(app):
+    text = (".. py:module:: example\n"
+            ".. py:class:: Class\n"
+            "\n"
+            "   :param age: blah blah\n"
+            "   :type age: int | str\n")
+    doctree = restructuredtext.parse(app, text)
+
+    assert_node(doctree,
+                (nodes.target,
+                 addnodes.index,
+                 addnodes.index,
+                 [desc, ([desc_signature, ([desc_annotation, "class "],
+                                           [desc_addname, "example."],
+                                           [desc_name, "Class"])],
+                         [desc_content, nodes.field_list, nodes.field, (nodes.field_name,
+                                                                        nodes.field_body)])]))
+    assert_node(doctree[3][1][0][0][1],
+                ([nodes.paragraph, ([addnodes.literal_strong, "age"],
+                                    " (",
+                                    [pending_xref, addnodes.literal_emphasis, "int"],
+                                    [addnodes.literal_emphasis, " | "],
+                                    [pending_xref, addnodes.literal_emphasis, "str"],
+                                    ")",
+                                    " -- ",
+                                    "blah blah")],))
+    assert_node(doctree[3][1][0][0][1][0][2], pending_xref,
+                refdomain="py", reftype="class", reftarget="int",
+                **{"py:module": "example", "py:class": "Class"})
+    assert_node(doctree[3][1][0][0][1][0][4], pending_xref,
+                refdomain="py", reftype="class", reftarget="str",
+                **{"py:module": "example", "py:class": "Class"})
+
+
 def test_info_field_list_var(app):
     text = (".. py:class:: Class\n"
             "\n"

</original_test_patch>

<candidate_a_patch>
diff --git a/sphinx/domains/python.py b/sphinx/domains/python.py
index 7d39d80ed..a78534647 100644
--- a/sphinx/domains/python.py
+++ b/sphinx/domains/python.py
@@ -349,6 +349,61 @@ class PyTypedField(PyXrefMixin, TypedField):
 
         return super().make_xref(rolename, domain, target, innernode, contnode, env)
 
+    def make_xrefs(self, rolename: str, domain: str, target: str,
+                   innernode: Type[TextlikeNode] = addnodes.literal_emphasis,
+                   contnode: Node = None, env: BuildEnvironment = None) -> List[Node]:
+        if '|' in target and env:
+            # Handle union types using | operator
+            try:
+                from sphinx.pycode.ast import parse as ast_parse
+                from sphinx.addnodes import desc_sig_punctuation
+
+                tree = ast_parse(target)
+                
+                def unparse(node):
+                    if isinstance(node, ast.BinOp):
+                        result = unparse(node.left)
+                        result.append(innernode('', ' | '))
+                        result.extend(unparse(node.right))
+                        return result
+                    elif isinstance(node, ast.Name):
+                        xref = type_to_xref(node.id, env)
+                        xref.clear()
+                        xref += innernode('', node.id)
+                        return [xref]
+                    elif isinstance(node, ast.Attribute):
+                        text = "%s.%s" % (unparse(node.value)[0], node.attr)
+                        xref = type_to_xref(text, env)
+                        xref.clear()
+                        xref += innernode('', text)
+                        return [xref]
+                    elif isinstance(node, ast.Subscript):
+                        result = unparse(node.value)
+                        result.append(addnodes.desc_sig_punctuation('', '['))
+                        result.extend(unparse(node.slice))
+                        result.append(addnodes.desc_sig_punctuation('', ']'))
+                        return result
+                    elif isinstance(node, ast.Index):
+                        return unparse(node.value)
+                    elif isinstance(node, ast.Tuple):
+                        result = []
+                        for elem in node.elts:
+                            result.extend(unparse(elem))
+                            result.append(addnodes.desc_sig_punctuation('', ', '))
+                        result.pop()
+                        return result
+                    elif isinstance(node, ast.Constant):
+                        return [nodes.Text(node.value)]
+                    elif isinstance(node, ast.Expr):
+                        return unparse(node.value)
+                    return [nodes.Text('')]
+                
+                result = unparse(tree.body[0].value)
+                return result
+            except SyntaxError:
+                pass
+        return super().make_xrefs(rolename, domain, target, innernode, contnode, env)
+
 
 class PyObject(ObjectDescription[Tuple[str, str]]):
     """

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sphinx/domains/python.py b/sphinx/domains/python.py
--- a/sphinx/domains/python.py
+++ b/sphinx/domains/python.py
@@ -304,7 +304,7 @@ def make_xref(self, rolename: str, domain: str, target: str,
     def make_xrefs(self, rolename: str, domain: str, target: str,
                    innernode: Type[TextlikeNode] = nodes.emphasis,
                    contnode: Node = None, env: BuildEnvironment = None) -> List[Node]:
-        delims = r'(\s*[\[\]\(\),](?:\s*or\s)?\s*|\s+or\s+|\.\.\.)'
+        delims = r'(\s*[\[\]\(\),](?:\s*or\s)?\s*|\s+or\s+|\s*\|\s*|\.\.\.)'
         delims_re = re.compile(delims)
         sub_targets = re.split(delims, target)
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_info_field_list_union_return_type",
  "specification_gap": "Union-member cross-referencing should apply consistently to return-type fields (`:rtype:`), not only parameter and variable type fields.",
  "input_description": "Parse a public `py:function` directive whose return type is `bytes | str`.",
  "expected_output": "The displayed return type remains `bytes | str`, while `bytes` and `str` become two separate Python class cross-references.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate B updates the shared Python field cross-reference behavior, including `:rtype:`. Candidate A overrides only `PyTypedField`, leaving return types as one incorrect cross-reference targeting the literal string `bytes | str`.",
  "test_patch": "diff --git a/tests/test_domain_py.py b/tests/test_domain_py.py\n--- a/tests/test_domain_py.py\n+++ b/tests/test_domain_py.py\n@@ -1007,6 +1007,20 @@ def test_info_field_list(app):\n     assert_node(doctree[3][1][0][0][1][0][3][0][6], pending_xref,\n                 refdomain=\"py\", reftype=\"class\", reftarget=\"str\",\n                 **{\"py:module\": \"example\", \"py:class\": \"Class\"})\n \n \n+def test_info_field_list_union_return_type(app):\n+    text = (\".. py:function:: decode()\\n\"\n+            \"\\n\"\n+            \"   :rtype: bytes | str\\n\")\n+    doctree = restructuredtext.parse(app, text)\n+\n+    xrefs = list(doctree.traverse(pending_xref))\n+    assert [(xref['reftarget'], xref.astext()) for xref in xrefs] == [\n+        ('bytes', 'bytes'),\n+        ('str', 'str'),\n+    ]\n+    assert 'bytes | str' in doctree.astext()\n+\n+\n def test_info_field_list_var(app):\n     text = (\".. py:class:: Class\\n\"\n             \"\\n\"\n",
  "test_command": "cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_union_return_type"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
/inputs/candidate.patch:19: trailing whitespace.
                
/inputs/candidate.patch:57: trailing whitespace.
                
warning: 2 lines add whitespace errors.
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
____________________ test_info_field_list_union_return_type ____________________

app = <SphinxTestApp buildername='html'>

    def test_info_field_list_union_return_type(app):
        text = (".. py:function:: decode()\n"
                "\n"
                "   :rtype: bytes | str\n")
        doctree = restructuredtext.parse(app, text)
    
        xrefs = list(doctree.traverse(pending_xref))
>       assert [(xref['reftarget'], xref.astext()) for xref in xrefs] == [
            ('bytes', 'bytes'),
            ('str', 'str'),
        ]
E       AssertionError: assert [('bytes | st...bytes | str')] == [('bytes', 'b...'str', 'str')]
E         
E         At index 0 diff: ('bytes | str', 'bytes | str') != ('bytes', 'bytes')
E         Right contains one more item: ('str', 'str')
E         Use -v to get more diff

tests/test_domain_py.py:1019: AssertionError
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
FAILED tests/test_domain_py.py::test_info_field_list_union_return_type - Asse...
1 failed, 7 warnings in 0.45s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
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
1 passed, 7 warnings in 0.40s
[pipeline] test_exit_code=0

```
</validated_execution>
