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
requests.get is ALWAYS sending content length
Hi,

It seems like that request.get always adds 'content-length' header to the request.
I think that the right behavior is not to add this header automatically in GET requests or add the possibility to not send it.

For example http://amazon.com returns 503 for every get request that contains 'content-length' header.

Thanks,

Oren


</issue_statement>

<original_test_patch>
diff --git a/test_requests.py b/test_requests.py
--- a/test_requests.py
+++ b/test_requests.py
@@ -58,6 +58,13 @@ def test_basic_building(self):
         assert pr.body == 'life=42'
 
 
+    def test_no_content_length(self):
+        get_req = requests.Request('GET', httpbin('get')).prepare()
+        self.assertTrue('Content-Length' not in get_req.headers)
+        head_req = requests.Request('HEAD', httpbin('head')).prepare()
+        self.assertTrue('Content-Length' not in head_req.headers)
+
+
     def test_path_is_not_double_encoded(self):
         request = requests.Request('GET', "http://0.0.0.0/get/test case").prepare()
 

</original_test_patch>

<candidate_a_patch>
diff --git a/requests/models.py b/requests/models.py
index 9926045..0c99e5d 100644
--- a/requests/models.py
+++ b/requests/models.py
@@ -386,7 +386,6 @@ class PreparedRequest(RequestEncodingMixin, RequestHooksMixin):
         self.body = body
 
     def prepare_content_length(self, body):
-        self.headers['Content-Length'] = '0'
         if hasattr(body, 'seek') and hasattr(body, 'tell'):
             body.seek(0, 2)
             self.headers['Content-Length'] = str(body.tell())

</candidate_a_patch>

<candidate_b_patch>
diff --git a/requests/models.py b/requests/models.py
--- a/requests/models.py
+++ b/requests/models.py
@@ -386,13 +386,14 @@ def prepare_body(self, data, files):
         self.body = body
 
     def prepare_content_length(self, body):
-        self.headers['Content-Length'] = '0'
         if hasattr(body, 'seek') and hasattr(body, 'tell'):
             body.seek(0, 2)
             self.headers['Content-Length'] = str(body.tell())
             body.seek(0, 0)
         elif body is not None:
             self.headers['Content-Length'] = str(len(body))
+        elif self.method not in ('GET', 'HEAD'):
+            self.headers['Content-Length'] = '0'
 
     def prepare_auth(self, auth):
         """Prepares the given HTTP auth data."""

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "bodyless_get_and_post_use_method_appropriate_framing",
  "specification_gap": "Suppressing the automatic Content-Length header is specific to bodyless retrieval methods such as GET and HEAD; it should not remove the existing explicit zero-length framing from body-capable methods such as POST.",
  "input_description": "Prepare two public requests with no supplied body: one GET and one POST to http://example.com/. No network request is performed.",
  "expected_output": "The prepared GET has no Content-Length header, while the prepared POST has Content-Length set to the string \"0\".",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests the method boundary implied by the issue rather than only its GET example. Candidate B conditionally omits the header for GET/HEAD and retains zero-length framing for POST, so it passes. Candidate A removes the default header for every method, so its prepared POST lacks Content-Length and fails.",
  "test_patch": "diff --git a/test_content_length_semantics.py b/test_content_length_semantics.py\nnew file mode 100644\n--- /dev/null\n+++ b/test_content_length_semantics.py\n@@ -0,0 +1,20 @@\n+import collections\n+import unittest\n+\n+try:\n+    from collections.abc import MutableMapping\n+except ImportError:\n+    from collections import MutableMapping\n+\n+collections.MutableMapping = MutableMapping\n+\n+import requests\n+\n+\n+class ContentLengthSemanticsTest(unittest.TestCase):\n+\n+    def test_bodyless_get_and_post_use_method_appropriate_framing(self):\n+        get_request = requests.Request('GET', 'http://example.com/').prepare()\n+        post_request = requests.Request('POST', 'http://example.com/').prepare()\n+        self.assertFalse('Content-Length' in get_request.headers)\n+        self.assertEqual(post_request.headers.get('Content-Length'), '0')\n",
  "test_command": "cd /testbed && python -m unittest test_content_length_semantics.ContentLengthSemanticsTest.test_bodyless_get_and_post_use_method_appropriate_framing"
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
/testbed/requests/models.py:559: SyntaxWarning: "is" with a literal. Did you mean "=="?
  if self.status_code is 0:
F
======================================================================
FAIL: test_bodyless_get_and_post_use_method_appropriate_framing (test_content_length_semantics.ContentLengthSemanticsTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/test_content_length_semantics.py", line 20, in test_bodyless_get_and_post_use_method_appropriate_framing
    self.assertEqual(post_request.headers.get('Content-Length'), '0')
AssertionError: None != '0'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
/testbed/requests/models.py:561: SyntaxWarning: "is" with a literal. Did you mean "=="?
  if self.status_code is 0:
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
