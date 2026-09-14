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
AdminSite.catch_all_view() drops query string in redirects
Description
	
#31747 introduced AdminSite.catch_all_view(). However, in the process it broke the ability to redirect with settings.APPEND_SLASH = True when there are query strings.
Provided URL: ​http://127.0.0.1:8000/admin/auth/foo?id=123
Expected redirect: ​http://127.0.0.1:8000/admin/auth/foo/?id=123
Actual redirect: ​http://127.0.0.1:8000/admin/auth/foo/
This seems to be because the redirect in question does not include the query strings (such as via request.META['QUERY_STRING']):
return HttpResponsePermanentRedirect("%s/" % request.path)
​https://github.com/django/django/blob/c57ff9ba5e251cd4c2761105a6046662c08f951e/django/contrib/admin/sites.py#L456

</issue_statement>

<original_test_patch>
diff --git a/tests/admin_views/tests.py b/tests/admin_views/tests.py
--- a/tests/admin_views/tests.py
+++ b/tests/admin_views/tests.py
@@ -8463,6 +8463,24 @@ def test_missing_slash_append_slash_true(self):
             response, known_url, status_code=301, target_status_code=403
         )
 
+    @override_settings(APPEND_SLASH=True)
+    def test_missing_slash_append_slash_true_query_string(self):
+        superuser = User.objects.create_user(
+            username="staff",
+            password="secret",
+            email="staff@example.com",
+            is_staff=True,
+        )
+        self.client.force_login(superuser)
+        known_url = reverse("admin:admin_views_article_changelist")
+        response = self.client.get("%s?id=1" % known_url[:-1])
+        self.assertRedirects(
+            response,
+            f"{known_url}?id=1",
+            status_code=301,
+            fetch_redirect_response=False,
+        )
+
     @override_settings(APPEND_SLASH=True)
     def test_missing_slash_append_slash_true_script_name(self):
         superuser = User.objects.create_user(
@@ -8481,6 +8499,24 @@ def test_missing_slash_append_slash_true_script_name(self):
             fetch_redirect_response=False,
         )
 
+    @override_settings(APPEND_SLASH=True)
+    def test_missing_slash_append_slash_true_script_name_query_string(self):
+        superuser = User.objects.create_user(
+            username="staff",
+            password="secret",
+            email="staff@example.com",
+            is_staff=True,
+        )
+        self.client.force_login(superuser)
+        known_url = reverse("admin:admin_views_article_changelist")
+        response = self.client.get("%s?id=1" % known_url[:-1], SCRIPT_NAME="/prefix/")
+        self.assertRedirects(
+            response,
+            f"/prefix{known_url}?id=1",
+            status_code=301,
+            fetch_redirect_response=False,
+        )
+
     @override_settings(APPEND_SLASH=True, FORCE_SCRIPT_NAME="/prefix/")
     def test_missing_slash_append_slash_true_force_script_name(self):
         superuser = User.objects.create_user(
@@ -8515,6 +8551,23 @@ def test_missing_slash_append_slash_true_non_staff_user(self):
             "/test_admin/admin/login/?next=/test_admin/admin/admin_views/article",
         )
 
+    @override_settings(APPEND_SLASH=True)
+    def test_missing_slash_append_slash_true_non_staff_user_query_string(self):
+        user = User.objects.create_user(
+            username="user",
+            password="secret",
+            email="user@example.com",
+            is_staff=False,
+        )
+        self.client.force_login(user)
+        known_url = reverse("admin:admin_views_article_changelist")
+        response = self.client.get("%s?id=1" % known_url[:-1])
+        self.assertRedirects(
+            response,
+            "/test_admin/admin/login/?next=/test_admin/admin/admin_views/article"
+            "%3Fid%3D1",
+        )
+
     @override_settings(APPEND_SLASH=False)
     def test_missing_slash_append_slash_false(self):
         superuser = User.objects.create_user(
@@ -8629,6 +8682,24 @@ def test_missing_slash_append_slash_true_without_final_catch_all_view(self):
             response, known_url, status_code=301, target_status_code=403
         )
 
+    @override_settings(APPEND_SLASH=True)
+    def test_missing_slash_append_slash_true_query_without_final_catch_all_view(self):
+        superuser = User.objects.create_user(
+            username="staff",
+            password="secret",
+            email="staff@example.com",
+            is_staff=True,
+        )
+        self.client.force_login(superuser)
+        known_url = reverse("admin10:admin_views_article_changelist")
+        response = self.client.get("%s?id=1" % known_url[:-1])
+        self.assertRedirects(
+            response,
+            f"{known_url}?id=1",
+            status_code=301,
+            fetch_redirect_response=False,
+        )
+
     @override_settings(APPEND_SLASH=False)
     def test_missing_slash_append_slash_false_without_final_catch_all_view(self):
         superuser = User.objects.create_user(

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/admin/sites.py b/django/contrib/admin/sites.py
index 61be31d890..051b742eeb 100644
--- a/django/contrib/admin/sites.py
+++ b/django/contrib/admin/sites.py
@@ -453,7 +453,10 @@ class AdminSite:
                 pass
             else:
                 if getattr(match.func, "should_append_slash", True):
-                    return HttpResponsePermanentRedirect("%s/" % request.path)
+                    new_path = request.path + "/"
+                    if request.META.get("QUERY_STRING"):
+                        new_path += "?" + request.META["QUERY_STRING"]
+                    return HttpResponsePermanentRedirect(new_path)
         raise Http404
 
     def _build_app_dict(self, request, label=None):

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/admin/sites.py b/django/contrib/admin/sites.py
--- a/django/contrib/admin/sites.py
+++ b/django/contrib/admin/sites.py
@@ -453,7 +453,9 @@ def catch_all_view(self, request, url):
                 pass
             else:
                 if getattr(match.func, "should_append_slash", True):
-                    return HttpResponsePermanentRedirect("%s/" % request.path)
+                    return HttpResponsePermanentRedirect(
+                        request.get_full_path(force_append_slash=True)
+                    )
         raise Http404
 
     def _build_app_dict(self, request, label=None):

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_missing_slash_append_slash_true_encoded_question_mark_and_query",
  "specification_gap": "The slash-appending admin redirect must preserve both query strings and the distinction between encoded path data and URL delimiters. A percent-encoded question mark in an object ID must remain %3F in the redirected path rather than becoming a literal query delimiter.",
  "input_description": "An authenticated staff user requests the slashless admin change URL for object ID \"question?mark\", encoded in the URL as question%3Fmark, with the query string source=search.",
  "expected_output": "A 301 response whose Location is /test_admin/admin/admin_views/article/question%3Fmark/change/?source=search. Candidate B escapes the path through get_full_path(); candidate A emits the decoded question mark literally and therefore produces a semantically different URL.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This covers a public URL invariant beyond merely retaining an ordinary query string. Candidate A's manual concatenation turns path data into URL structure, while candidate B preserves the original path/query boundary.",
  "test_patch": "diff --git a/tests/admin_views/tests.py b/tests/admin_views/tests.py\n--- a/tests/admin_views/tests.py\n+++ b/tests/admin_views/tests.py\n@@ -8462,6 +8462,21 @@ class AdminSiteFinalCatchAllPatternTests(TestCase):\n         self.assertRedirects(\n             response, known_url, status_code=301, target_status_code=403\n         )\n \n+    @override_settings(APPEND_SLASH=True)\n+    def test_missing_slash_append_slash_true_encoded_question_mark_and_query(self):\n+        staff_user = User.objects.create_user(\n+            username=\"staff\",\n+            password=\"secret\",\n+            is_staff=True,\n+        )\n+        self.client.force_login(staff_user)\n+        known_url = reverse(\n+            \"admin:admin_views_article_change\", args=(\"question?mark\",)\n+        )\n+        response = self.client.get(known_url[:-1], {\"source\": \"search\"})\n+        self.assertEqual(response.status_code, 301)\n+        self.assertEqual(response[\"Location\"], f\"{known_url}?source=search\")\n+\n     @override_settings(APPEND_SLASH=True)\n     def test_missing_slash_append_slash_true_script_name(self):\n         superuser = User.objects.create_user(\n",
  "test_command": "cd /testbed && python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_encoded_question_mark_and_query --verbosity 0"
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
System check identified no issues (1 silenced).
======================================================================
FAIL: test_missing_slash_append_slash_true_encoded_question_mark_and_query (admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_encoded_question_mark_and_query)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/admin_views/tests.py", line 8479, in test_missing_slash_append_slash_true_encoded_question_mark_and_query
    self.assertEqual(response["Location"], f"{known_url}?source=search")
AssertionError: '/test_admin/admin/admin_views/article/question?mark/change/?source=search' != '/test_admin/admin/admin_views/article/question%3Fmark/change/?source=search'
- /test_admin/admin/admin_views/article/question?mark/change/?source=search
?                                               ^
+ /test_admin/admin/admin_views/article/question%3Fmark/change/?source=search
?                                               ^^^


----------------------------------------------------------------------
Ran 1 test in 0.208s

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
System check identified no issues (1 silenced).
----------------------------------------------------------------------
Ran 1 test in 0.207s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
