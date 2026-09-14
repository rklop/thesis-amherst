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
AuthenticationForm's username field doesn't set maxlength HTML attribute.
Description
	
AuthenticationForm's username field doesn't render with maxlength HTML attribute anymore.
Regression introduced in #27515 and 5ceaf14686ce626404afb6a5fbd3d8286410bf13.
​https://groups.google.com/forum/?utm_source=digest&utm_medium=email#!topic/django-developers/qnfSqro0DlA
​https://forum.djangoproject.com/t/possible-authenticationform-max-length-regression-in-django-2-1/241

</issue_statement>

<original_test_patch>
diff --git a/tests/auth_tests/test_forms.py b/tests/auth_tests/test_forms.py
--- a/tests/auth_tests/test_forms.py
+++ b/tests/auth_tests/test_forms.py
@@ -423,6 +423,7 @@ def test_username_field_max_length_matches_user_model(self):
         CustomEmailField.objects.create_user(**data)
         form = AuthenticationForm(None, data)
         self.assertEqual(form.fields['username'].max_length, 255)
+        self.assertEqual(form.fields['username'].widget.attrs.get('maxlength'), 255)
         self.assertEqual(form.errors, {})
 
     @override_settings(AUTH_USER_MODEL='auth_tests.IntegerUsernameUser')
@@ -435,6 +436,7 @@ def test_username_field_max_length_defaults_to_254(self):
         IntegerUsernameUser.objects.create_user(**data)
         form = AuthenticationForm(None, data)
         self.assertEqual(form.fields['username'].max_length, 254)
+        self.assertEqual(form.fields['username'].widget.attrs.get('maxlength'), 254)
         self.assertEqual(form.errors, {})
 
     def test_username_field_label(self):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/auth/forms.py b/django/contrib/auth/forms.py
index a0cfed0995..f6f73ba3e8 100644
--- a/django/contrib/auth/forms.py
+++ b/django/contrib/auth/forms.py
@@ -192,6 +192,7 @@ class AuthenticationForm(forms.Form):
         # Set the max length and label for the "username" field.
         self.username_field = UserModel._meta.get_field(UserModel.USERNAME_FIELD)
         self.fields['username'].max_length = self.username_field.max_length or 254
+        self.fields['username'].widget.attrs['maxlength'] = self.fields['username'].max_length
         if self.fields['username'].label is None:
             self.fields['username'].label = capfirst(self.username_field.verbose_name)
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/auth/forms.py b/django/contrib/auth/forms.py
--- a/django/contrib/auth/forms.py
+++ b/django/contrib/auth/forms.py
@@ -191,7 +191,9 @@ def __init__(self, request=None, *args, **kwargs):
 
         # Set the max length and label for the "username" field.
         self.username_field = UserModel._meta.get_field(UserModel.USERNAME_FIELD)
-        self.fields['username'].max_length = self.username_field.max_length or 254
+        username_max_length = self.username_field.max_length or 254
+        self.fields['username'].max_length = username_max_length
+        self.fields['username'].widget.attrs['maxlength'] = username_max_length
         if self.fields['username'].label is None:
             self.fields['username'].label = capfirst(self.username_field.verbose_name)
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_custom_username_field_effective_max_length_in_html",
  "specification_gap": "For an AuthenticationForm subclass with a custom username field that normalizes assigned maximum lengths, the rendered HTML maxlength should reflect the form field's effective validation limit, not the unnormalized user-model value.",
  "input_description": "Instantiate a custom AuthenticationForm whose username CharField caps every assigned max_length at 32. The default user model supplies 150, while the custom field retains an effective limit and validator of 32.",
  "expected_output": "The rendered username input contains maxlength=\"32\". Candidate A reads the effective form-field value after assignment; Candidate B instead renders the captured model value, maxlength=\"150\".",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises AuthenticationForm's documented subclassing surface and checks the externally visible invariant that browser-side maxlength agrees with the customized form field's effective server-side limit. It isolates the only semantic difference between the otherwise equivalent candidates.",
  "test_patch": "diff --git a/tests/auth_tests/test_forms.py b/tests/auth_tests/test_forms.py\n--- a/tests/auth_tests/test_forms.py\n+++ b/tests/auth_tests/test_forms.py\n@@ -424,6 +424,30 @@ class AuthenticationFormTest(TestDataMixin, TestCase):\n         form = AuthenticationForm(None, data)\n         self.assertEqual(form.fields['username'].max_length, 255)\n         self.assertEqual(form.errors, {})\n+\n+    def test_custom_username_field_effective_max_length_in_html(self):\n+        class CappedCharField(CharField):\n+            def __init__(self, *, max_length, **kwargs):\n+                self.max_allowed_length = max_length\n+                super().__init__(max_length=max_length, **kwargs)\n+\n+            @property\n+            def max_length(self):\n+                return self._max_length\n+\n+            @max_length.setter\n+            def max_length(self, value):\n+                if value is None:\n+                    self._max_length = None\n+                else:\n+                    self._max_length = min(value, self.max_allowed_length)\n+\n+        class CustomAuthenticationForm(AuthenticationForm):\n+            username = CappedCharField(max_length=32)\n+\n+        form = CustomAuthenticationForm()\n+        self.assertEqual(form.fields['username'].max_length, 32)\n+        self.assertIn('maxlength=\"32\"', str(form['username']))\n \n     @override_settings(AUTH_USER_MODEL='auth_tests.IntegerUsernameUser')\n     def test_username_field_max_length_defaults_to_254(self):\n",
  "test_command": "python tests/runtests.py auth_tests.test_forms.AuthenticationFormTest.test_custom_username_field_effective_max_length_in_html"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.012s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_custom_username_field_effective_max_length_in_html (auth_tests.test_forms.AuthenticationFormTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/auth_tests/test_forms.py", line 450, in test_custom_username_field_effective_max_length_in_html
    self.assertIn('maxlength="32"', str(form['username']))
AssertionError: 'maxlength="32"' not found in '<input type="text" name="username" maxlength="150" required id="id_username">'

----------------------------------------------------------------------
Ran 1 test in 0.012s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```
</validated_execution>
