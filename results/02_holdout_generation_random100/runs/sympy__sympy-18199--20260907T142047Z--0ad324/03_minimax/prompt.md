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
nthroot_mod function misses one root of x = 0 mod p.
When in the equation x**n = a mod p , when a % p == 0. Then x = 0 mod p is also a root of this equation. But right now `nthroot_mod` does not check for this condition. `nthroot_mod(17*17, 5 , 17)` has a root `0 mod 17`. But it does not return it.

</issue_statement>

<original_test_patch>
diff --git a/sympy/ntheory/tests/test_residue.py b/sympy/ntheory/tests/test_residue.py
--- a/sympy/ntheory/tests/test_residue.py
+++ b/sympy/ntheory/tests/test_residue.py
@@ -162,7 +162,8 @@ def test_residue():
     assert is_nthpow_residue(31, 4, 41)
     assert not is_nthpow_residue(2, 2, 5)
     assert is_nthpow_residue(8547, 12, 10007)
-    raises(NotImplementedError, lambda: nthroot_mod(29, 31, 74))
+
+    assert nthroot_mod(29, 31, 74) == [45]
     assert nthroot_mod(1801, 11, 2663) == 44
     for a, q, p in [(51922, 2, 203017), (43, 3, 109), (1801, 11, 2663),
           (26118163, 1303, 33333347), (1499, 7, 2663), (595, 6, 2663),
@@ -170,8 +171,12 @@ def test_residue():
         r = nthroot_mod(a, q, p)
         assert pow(r, q, p) == a
     assert nthroot_mod(11, 3, 109) is None
-    raises(NotImplementedError, lambda: nthroot_mod(16, 5, 36))
-    raises(NotImplementedError, lambda: nthroot_mod(9, 16, 36))
+    assert nthroot_mod(16, 5, 36, True) == [4, 22]
+    assert nthroot_mod(9, 16, 36, True) == [3, 9, 15, 21, 27, 33]
+    assert nthroot_mod(4, 3, 3249000) == []
+    assert nthroot_mod(36010, 8, 87382, True) == [40208, 47174]
+    assert nthroot_mod(0, 12, 37, True) == [0]
+    assert nthroot_mod(0, 7, 100, True) == [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
 
     for p in primerange(5, 100):
         qv = range(3, p, 4)
diff --git a/sympy/solvers/tests/test_solveset.py b/sympy/solvers/tests/test_solveset.py
--- a/sympy/solvers/tests/test_solveset.py
+++ b/sympy/solvers/tests/test_solveset.py
@@ -2242,11 +2242,12 @@ def test_solve_modular():
     assert solveset(Mod(3**(3**x), 4) - 3, x, S.Integers) == \
             Intersection(ImageSet(Lambda(n, Intersection({log(2*n + 1)/log(3)},
             S.Integers)), S.Naturals0), S.Integers)
-    # Not Implemented for m without primitive root
+    # Implemented for m without primitive root
     assert solveset(Mod(x**3, 8) - 1, x, S.Integers) == \
-            ConditionSet(x, Eq(Mod(x**3, 8) - 1, 0), S.Integers)
+            ImageSet(Lambda(n, 8*n + 1), S.Integers)
     assert solveset(Mod(x**4, 9) - 4, x, S.Integers) == \
-            ConditionSet(x, Eq(Mod(x**4, 9) - 4, 0), S.Integers)
+            Union(ImageSet(Lambda(n, 9*n + 4), S.Integers),
+            ImageSet(Lambda(n, 9*n + 5), S.Integers))
     # domain intersection
     assert solveset(3 - Mod(5*x - 8, 7), x, S.Naturals0) == \
             Intersection(ImageSet(Lambda(n, 7*n + 5), S.Integers), S.Naturals0)

</original_test_patch>

<candidate_a_patch>
diff --git a/sympy/ntheory/residue_ntheory.py b/sympy/ntheory/residue_ntheory.py
index 9a3052d4c3..c838c1c748 100644
--- a/sympy/ntheory/residue_ntheory.py
+++ b/sympy/ntheory/residue_ntheory.py
@@ -767,46 +767,211 @@ def nthroot_mod(a, n, p, all_roots=False):
     23
     """
     from sympy.core.numbers import igcdex
+    from sympy.polys.galoistools import gf_crt1, gf_crt2
+    from sympy.polys.domains import ZZ
+
     a, n, p = as_int(a), as_int(n), as_int(p)
     if n == 2:
         return sqrt_mod(a, p, all_roots)
-    # see Hackman "Elementary Number Theory" (2009), page 76
-    if not is_nthpow_residue(a, n, p):
-        return None
-    if not isprime(p):
-        raise NotImplementedError("Not implemented for composite p")
+
+    # Handle the case a % p == 0
+    a_mod = a % p
+    if a_mod == 0:
+        return _nthroot_mod0(a_mod, n, p, all_roots)
+
+    if isprime(p):
+        # Check if a solution exists for prime modulus
+        if not is_nthpow_residue(a, n, p):
+            return None
+        return _nthroot_mod_prime(a_mod, n, p, all_roots)
+    else:
+        # For composite modulus, try to find solutions using CRT
+        # Don't use is_nthpow_residue as it may give false negatives
+        # Always return a list for composite moduli
+        return _nthroot_mod_composite(a_mod, n, p, all_roots)
+
+
+def _nthroot_mod0(a, n, p, all_roots):
+    """
+    Find solutions to x^n = a mod p when a % p == 0.
+    """
+    from sympy.polys.galoistools import gf_crt1, gf_crt2
+    from sympy.polys.domains import ZZ
+
+    if isprime(p):
+        # For prime p, x^n = 0 mod p has only solution x = 0
+        return [0] if all_roots else 0
+
+    # For composite modulus, solve for each prime power
+    f = factorint(p)
+    v = []
+    pv = []
+    for px, ex in f.items():
+        pk = pow(px, ex)
+        # Solve x^n = 0 mod p^k
+        roots = _nthroot_mod_prime_power(0, n, px, ex)
+        if roots is None:
+            return []
+        v.append(roots)
+        pv.append(pk)
+
+    # Combine solutions using CRT
+    mm, e, s = gf_crt1(pv, ZZ)
+    res = []
+    for vx in _product(*v):
+        r = gf_crt2(vx, pv, mm, e, s, ZZ)
+        res.append(r)
+    res = sorted(set(res))
+    if all_roots:
+        return res
+    else:
+        return res
+
+
+def _nthroot_mod_prime(a, n, p, all_roots):
+    """
+    Find solutions to x^n = a mod p where p is prime.
+    """
+    from sympy.core.numbers import igcdex
 
     if (p - 1) % n == 0:
         return _nthroot_mod1(a, n, p, all_roots)
+
     # The roots of ``x**n - a = 0 (mod p)`` are roots of
     # ``gcd(x**n - a, x**(p - 1) - 1) = 0 (mod p)``
     pa = n
     pb = p - 1
     b = 1
+    a_mod = a
     if pa < pb:
-        a, pa, b, pb = b, pb, a, pa
+        a_mod, pa, b, pb = b, pb, a_mod, pa
     while pb:
-        # x**pa - a = 0; x**pb - b = 0
-        # x**pa - a = x**(q*pb + r) - a = (x**pb)**q * x**r - a =
-        #             b**q * x**r - a; x**r - c = 0; c = b**-q * a mod p
         q, r = divmod(pa, pb)
         c = pow(b, q, p)
         c = igcdex(c, p)[0]
-        c = (c * a) % p
+        c = (c * a_mod) % p
         pa, pb = pb, r
-        a, b = b, c
+        a_mod, b = b, c
     if pa == 1:
         if all_roots:
-            res = [a]
+            res = [a_mod]
         else:
-            res = a
+            res = a_mod
     elif pa == 2:
-        return sqrt_mod(a, p , all_roots)
+        return sqrt_mod(a_mod, p, all_roots)
     else:
-        res = _nthroot_mod1(a, pa, p, all_roots)
+        res = _nthroot_mod1(a_mod, pa, p, all_roots)
+    return res
+
+
+def _nthroot_mod_composite(a, n, p, all_roots):
+    """
+    Find solutions to x^n = a mod p where p is composite.
+    Uses Chinese Remainder Theorem to combine solutions from prime powers.
+    """
+    from sympy.polys.galoistools import gf_crt1, gf_crt2
+    from sympy.polys.domains import ZZ
+
+    f = factorint(p)
+    v = []
+    pv = []
+    for px, ex in f.items():
+        pk = pow(px, ex)
+        roots = _nthroot_mod_prime_power(a % pk, n, px, ex)
+        if roots is None:
+            return []
+        if not roots:
+            return []
+        v.append(roots)
+        pv.append(pk)
+
+    # Combine solutions using CRT
+    mm, e, s = gf_crt1(pv, ZZ)
+    res = []
+    for vx in _product(*v):
+        r = gf_crt2(vx, pv, mm, e, s, ZZ)
+        res.append(r)
+    
+    res = sorted(set(res))
     return res
 
 
+def _nthroot_mod_prime_power(a, n, p, k):
+    """
+    Find solutions to x^n = a mod p^k where p is prime.
+    Returns None if no solution exists.
+    """
+    from sympy.core.numbers import igcdex
+
+    pk = pow(p, k)
+    a = a % pk
+
+    # Handle a == 0
+    if a == 0:
+        # x^n = 0 mod p^k means p^(ceil(k/n)) divides x
+        # Solutions are multiples of p^ceil(k/n) in [0, p^k)
+        min_exp = (k + n - 1) // n  # ceil(k/n)
+        step = pow(p, min_exp)
+        roots = []
+        for i in range(pow(p, k - min_exp)):
+            roots.append(i * step)
+        return roots
+
+    # For a != 0 mod p, check if solution exists mod p
+    if not is_nthpow_residue(a, n, p):
+        return None
+
+    # Get all solutions mod p
+    roots_mod_p = _nthroot_mod_prime(a % p, n, p, True)
+    if roots_mod_p is None:
+        return None
+
+    # For each root mod p, lift to mod p^k
+    all_roots = []
+    for r in roots_mod_p:
+        lifted = _lift_nthroot(r, a, n, p, k)
+        if lifted:
+            all_roots.extend(lifted)
+    
+    # Remove duplicates
+    all_roots = sorted(set(all_roots))
+    return all_roots if all_roots else None
+
+
+def _lift_nthroot(r, a, n, p, k):
+    """
+    Lift a solution r of x^n = a mod p to a solution mod p^k.
+    """
+    from sympy.core.numbers import igcdex
+
+    # Reduce a modulo p^k
+    pk = pow(p, k)
+    a = a % pk
+
+    # We already have solution r mod p
+    # Need to lift to higher powers
+    # Use iterative approach: find solutions at each power p^i
+
+    current_roots = [r % p]
+    for i in range(1, k):
+        p_i = pow(p, i)
+        p_next = pow(p, i + 1)
+        new_roots = []
+        for r_i in current_roots:
+            # Try all possible deltas - simpler approach
+            for delta in range(p):
+                new_r = (r_i + delta * p_i) % p_next
+                if pow(new_r, n, p_next) == a % p_next:
+                    if new_r not in new_roots:
+                        new_roots.append(new_r)
+
+        if not new_roots:
+            return None
+        current_roots = new_roots
+
+    return current_roots
+
+
 def quadratic_residues(p):
     """
     Returns the list of quadratic residues.

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sympy/ntheory/residue_ntheory.py b/sympy/ntheory/residue_ntheory.py
--- a/sympy/ntheory/residue_ntheory.py
+++ b/sympy/ntheory/residue_ntheory.py
@@ -2,6 +2,7 @@
 
 from sympy.core.compatibility import as_int, range
 from sympy.core.function import Function
+from sympy.utilities.iterables import cartes
 from sympy.core.numbers import igcd, igcdex, mod_inverse
 from sympy.core.power import isqrt
 from sympy.core.singleton import S
@@ -742,6 +743,48 @@ def _nthroot_mod1(s, q, p, all_roots):
         return res
     return min(res)
 
+def _nthroot_mod_composite(a, n, m):
+    """
+    Find the solutions to ``x**n = a mod m`` when m is not prime.
+    """
+    from sympy.ntheory.modular import crt
+    f = factorint(m)
+    dd = {}
+    for p, e in f.items():
+        tot_roots = set()
+        if e == 1:
+            tot_roots.update(nthroot_mod(a, n, p, True) or [])
+        else:
+            for root in nthroot_mod(a, n, p, True) or []:
+                rootn = pow(root, n)
+                diff = (rootn // (root or 1) * n) % p
+                if diff != 0:
+                    ppow = p
+                    for j in range(1, e):
+                        ppow *= p
+                        root = (root - (rootn - a) * mod_inverse(diff, p)) % ppow
+                    tot_roots.add(root)
+                else:
+                    new_base = p
+                    roots_in_base = {root}
+                    while new_base < pow(p, e):
+                        new_base *= p
+                        new_roots = set()
+                        for k in roots_in_base:
+                            if (pow(k, n) - a) % (new_base) != 0:
+                                continue
+                            while k not in new_roots:
+                                new_roots.add(k)
+                                k = (k + (new_base // p)) % new_base
+                        roots_in_base = new_roots
+                    tot_roots = tot_roots | roots_in_base
+        dd[pow(p, e)] = tot_roots
+    a = []
+    m = []
+    for x, y in dd.items():
+        m.append(x)
+        a.append(list(y))
+    return sorted(set(crt(m, list(i))[0] for i in cartes(*a)))
 
 def nthroot_mod(a, n, p, all_roots=False):
     """
@@ -771,11 +814,12 @@ def nthroot_mod(a, n, p, all_roots=False):
     if n == 2:
         return sqrt_mod(a, p, all_roots)
     # see Hackman "Elementary Number Theory" (2009), page 76
+    if not isprime(p):
+        return _nthroot_mod_composite(a, n, p)
+    if a % p == 0:
+        return [0]
     if not is_nthpow_residue(a, n, p):
         return None
-    if not isprime(p):
-        raise NotImplementedError("Not implemented for composite p")
-
     if (p - 1) % n == 0:
         return _nthroot_mod1(a, n, p, all_roots)
     # The roots of ``x**n - a = 0 (mod p)`` are roots of

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_nthroot_mod_zero_residue_at_prime_factor",
  "specification_gap": "A zero residue must be handled at each prime-power factor of a composite modulus, even when the radicand is nonzero modulo the complete modulus.",
  "input_description": "Call nthroot_mod(7**3, 3, 7**4, all_roots=True). Here the radicand is nonzero modulo 2401 but is zero modulo its prime factor 7.",
  "expected_output": "The sorted 147 roots in [0, 2401), precisely the integers congruent to 7, 14, or 28 modulo 49.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This extends the reported zero-residue boundary through the public composite-modulus entry point. Candidate B routes the zero residue modulo 7 through nthroot_mod and lifts it; candidate A passes zero directly to its nonzero prime-root helper, which attempts an invalid discrete logarithm and raises ValueError.",
  "test_patch": "diff --git a/sympy/ntheory/tests/test_residue.py b/sympy/ntheory/tests/test_residue.py\n--- a/sympy/ntheory/tests/test_residue.py\n+++ b/sympy/ntheory/tests/test_residue.py\n@@ -244,3 +244,10 @@ def test_residue():\n     args = 5779, 3528, 6215\n     assert discrete_log(*args) == 687\n     assert discrete_log(*Tuple(*args)) == 687\n+\n+\n+def test_nthroot_mod_zero_residue_at_prime_factor():\n+    p = 7\n+    modulus = p**4\n+    expected = sorted(r + 49*k for r in (7, 14, 28) for k in range(49))\n+    assert nthroot_mod(p**3, 3, modulus, True) == expected\n",
  "test_command": "cd /testbed && python -c 'from sympy.ntheory.tests.test_residue import test_nthroot_mod_zero_residue_at_prime_factor; test_nthroot_mod_zero_residue_at_prime_factor()'"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
/inputs/candidate.patch:148: trailing whitespace.
    
/inputs/candidate.patch:189: trailing whitespace.
    
warning: 2 lines add whitespace errors.
[pipeline] checking generated test patch
[pipeline] executing generated test command
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/testbed/sympy/ntheory/tests/test_residue.py", line 253, in test_nthroot_mod_zero_residue_at_prime_factor
    assert nthroot_mod(p**3, 3, modulus, True) == expected
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 791, in nthroot_mod
    return _nthroot_mod_composite(a_mod, n, p, all_roots)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 880, in _nthroot_mod_composite
    roots = _nthroot_mod_prime_power(a % pk, n, px, ex)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 925, in _nthroot_mod_prime_power
    roots_mod_p = _nthroot_mod_prime(a % p, n, p, True)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 838, in _nthroot_mod_prime
    return _nthroot_mod1(a, n, p, all_roots)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 728, in _nthroot_mod1
    t = discrete_log(p, s1, h)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 1461, in discrete_log
    return _discrete_log_trial_mul(n, a, b, order)
  File "/testbed/sympy/ntheory/residue_ntheory.py", line 1221, in _discrete_log_trial_mul
    raise ValueError("Log does not exist")
ValueError: Log does not exist
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
[pipeline] test_exit_code=0

```
</validated_execution>
