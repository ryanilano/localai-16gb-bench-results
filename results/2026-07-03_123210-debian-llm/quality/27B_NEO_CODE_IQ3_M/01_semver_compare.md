# 27B_NEO_CODE_IQ3_M — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process:

1.  **Understand the Goal**: The user wants a Python function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings according to the official precedence rules. It should return -1, 0, or 1. I also need to provide 6 unit tests covering specific edge cases.

2.  **Analyze SemVer Precedence Rules**:
   - Format: `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`
   - Precedence:
     - Compare MAJOR, MINOR, PATCH numerically.
     - A version with a pre-release tag has LOWER precedence than the same version without one.
     - Pre-release identifiers are compared dot-by-dot:
       - Numeric identifiers are compared as integers.
       - Alphanumeric identifiers are compared lexically in ASCII order.
       - Numeric identifiers always have lower precedence than alphanumeric identifiers.
       - If all identifiers up to the point of differentiation are equal, the version with more identifiers has higher precedence.
     - Build metadata (`+...`) is ignored for precedence.

3.  **Design the Function**:
   - Parse the version string into components: major, minor, patch, pre-release, build metadata.
   - Regular expression is useful: `^(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$`
     - Actually, I can simplify parsing since I just need to split by `-` and `+` carefully. But SemVer spec says pre-release can contain `-`, so splitting by `-` directly is tricky. Better to use regex or careful string splitting.
     - Simpler approach: Split by `+` first to ignore build metadata. Then split the rest by `-` to separate core version from pre-release. The pre-release part is everything after the first `-` in the remaining string.
     - Let's verify: `1.0.0-alpha.beta+build.123` -> split by `+` -> `1.0.0-alpha.beta`, ignore rest.
     - Split `1.0.0-alpha.beta` by `-` -> `1.0.0`, `alpha.beta`.
     - Core version: split by `.` -> major, minor, patch.
     - Pre-release: split by `.` -> list of identifiers.
   - Comparison logic:
     - Compare major, minor, patch numerically. If different, return result.
     - If core versions are equal:
       - If both have no pre-release: return 0.
       - If `a` has pre-release and `b` doesn't: `a < b` -> return -1.
       - If `b` has pre-release and `a` doesn't: `a > b` -> return 1.
       - If both have pre-release: compare identifiers dot-by-dot.
         - Iterate through identifiers of both (up to max length).
         - For each pair:
           - Check if numeric.
           - Rules:
             - Both numeric: compare as ints.
             - Both alphanumeric: compare as strings.
             - One numeric, one alphanumeric: numeric < alphanumeric.
           - If different, return result.
         - If all compared identifiers are equal, the one with more identifiers is greater.
   - Edge cases to handle in parsing:
     - Pre-release identifiers can be numeric or alphanumeric.
     - Numeric identifiers: only contain digits `[0-9]+`.
     - Alphanumeric: contain at least one non-digit `[a-zA-Z0-9-]*[a-zA-Z-][a-zA-Z0-9-]*` (but for comparison, just checking `isdigit()` is usually enough for SemVer 2.0.0, as identifiers are either all digits or contain non-digits).
     - Wait, SemVer spec says numeric identifiers must NOT have leading zeros, except for `0`. But for comparison, we can just parse them as ints or strings. The spec says "Numeric identifiers are compared as integers." So `int()` is fine.

4.  **Draft the Code (Mental Refinement)**:
   ```python
   import re

   def compare_versions(a: str, b: str) -> int:
       # Helper to parse version string
       def parse_version(v):
           # Remove build metadata
           v = v.split('+')[0]
           # Split into core and pre-release
           if '-' in v:
               core, pre = v.split('-', 1)
               pre_parts = pre.split('.')
           else:
               core, pre_parts = v, None

           major, minor, patch = map(int, core.split('.'))
           return major, minor, patch, pre_parts

       maj_a, min_a, pat_a, pre_a = parse_version(a)
       maj_b, min_b, pat_b, pre_b = parse_version(b)

       # Compare core
       if (maj_a, min_a, pat_a) != (maj_b, min_b, pat_b):
           return 1 if (maj_a, min_a, pat_a) > (maj_b, min_b, pat_b) else -1

       # Both core versions are equal
       if pre_a is None and pre_b is None:
           return 0
       if pre_a is not None and pre_b is None:
           return -1
       if pre_a is None and pre_b is not None:
           return 1

       # Both have pre-release
       # Compare dot-by-dot
       len_a = len(pre_a)
       len_b = len(pre_b)
       min_len = min(len_a, len_b)

       for i in range(min_len):
           id_a = pre_a[i]
           id_b = pre_b[i]

           is_num_a = id_a.isdigit()
           is_num_b = id_b.isdigit()

           if is_num_a and is_num_b:
               # Both numeric
               val_a = int(id_a)
               val_b = int(id_b)
               if val_a != val_b:
                   return 1 if val_a > val_b else -1
           elif is_num_a and not is_num_b:
               # Numeric < Alphanumeric
               return -1
           elif not is_num_a and is_num_b:
               # Alphanumeric > Numeric
               return 1
           else:
               # Both alphanumeric
               if id_a != id_b:
                   return 1 if id_a > id_b else -1

       # All compared identifiers are equal, longer list is greater
       if len_a != len_b:
           return 1 if len_a > len_b else -1

       return 0
   ```

5.  **Check against SemVer Spec**:
   - "Numeric identifiers are compared as integers." -> `int()` works.
   - "Alphanumeric identifiers are compared lexically in ASCII sort order." -> string comparison works.
   - "Numeric identifiers always have lower precedence than alphanumeric identifiers." -> handled.
   - "A larger set of fields always has higher precedence than a smaller set if all preceding identifiers are equal." -> handled by length check at the end.
   - "A version with a pre-release tag has lower precedence than the same version without one." -> handled.
   - Build metadata ignored -> handled by `split('+')[0]`.
   - Wait, what about identifiers that are purely numeric but have leading zeros? SemVer says they are invalid, but if they occur, `isdigit()` still works, and `int()` parses them. The spec says identifiers are either all digits or contain non-digits. `isdigit()` is fine.
   - One minor thing: `id_a.isdigit()` might return True for unicode digits, but SemVer is ASCII. I'll stick with `id_a.isdecimal()` or just `id_a.isdigit()`. It's fine. Alternatively, `id_a[0].isdigit()` might be safer if we consider negative numbers? No, SemVer pre-release identifiers don't have signs. So `isdigit()` is fine.
   - Actually, to be strictly compliant with ASCII lexical order, string comparison is fine.
   - Let's test mentally: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`
     - `pre_a = ['alpha', '1']`, `pre_b = ['alpha', 'beta']`
     - i=0: both 'alpha', equal.
     - i=1: '1' is numeric, 'beta' is alphanumeric. Numeric < Alphanumeric -> returns -1. Correct.
   - `1.0.0-alpha` vs `1.0.0-alpha.1`
     - i=0: equal.
     - loop ends. len_a=1, len_b=2. len_a < len_b -> returns -1. Correct.
   - `1.0.0` vs `1.0.0-alpha`
     - pre_a=None, pre_b=['alpha'] -> returns 1. Correct.
   - Build metadata: `1.0.0+build1` vs `1.0.0+build2` -> both parse to same core, pre=None -> returns 0. Correct.

6.  **Unit Tests**:
   Need 6 tests covering:
   1. Pre-release vs release (`1.0.0-alpha` vs `1.0.0`)
   2. Numeric vs alphanumeric identifiers (`1.0.0-alpha.1` vs `1.0.0-alpha.beta`)
   3. Differing field counts (`1.0.0-alpha` vs `1.0.0-alpha.1`)
   4. Build metadata ignored (`1.0.0+build1` vs `1.0.0+build2`)
   5. Core version comparison (`1.0.0` vs `2.0.0`)
   6. Alphanumeric lexical comparison (`1.0.0-alpha` vs `1.0.0-beta`)

   I'll write them using `assert` or `unittest`. The prompt says "Include 6 unit tests", I'll use simple `assert` statements or `unittest.TestCase`. I'll stick to `assert` for brevity, or maybe a `if __name__ == "__main__":` block with assertions. I'll format them clearly.

   Let's draft the tests:
   ```python
   assert compare_versions("1.0.0-alpha", "1.0.0") == -1
   assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1
   assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1
   assert compare_versions("1.0.0+build1", "1.0.0+build2") == 0
   assert compare_versions("1.0.0", "2.0.0") == -1
   assert compare_versions("1.0.0-alpha", "1.0.0-beta") == -1
   ```
   Wait, the prompt says "specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored." That's 4 specific cases. I'll add two more for completeness, or just make sure those 4 are explicitly tested, plus maybe core version comparison and equal versions.

   I'll structure the tests clearly.

7.  **Refine Code & Edge Cases**:
   - What about `isdigit()` vs `isnumeric()`? In Python, `isdigit()` works for ASCII digits. It's fine.
   - What if an identifier is like `01`? SemVer says invalid, but `int('01')` is `1`. It's fine.
   - What about empty pre-release? The spec says pre-release can't be empty. We can assume valid SemVer strings as per typical coding challenges, but I'll add a small safeguard or just assume valid input. The prompt says "compares two Semantic Versioning 2.0.0 strings", implying valid inputs.
   - I should handle the case where `split('+')` might not have a second part. `v.split('+')[0]` is safe.
   - I'll write the function cleanly.

   Let's double check the comparison logic for alphanumeric vs numeric.
   Spec: "Numeric identifiers always have lower precedence than alphanumeric identifiers."
   My code:
   ```python
           if is_num_a and is_num_b:
               val_a, val_b = int(id_a), int(id_b)
               if val_a != val_b: return 1 if val_a > val_b else -1
           elif is_num_a and not is_num_b:
               return -1
           elif not is_num_a and is_num_b:
               return 1
           else:
               if id_a != id_b: return 1 if id_a > id_b else -1
   ```
   This matches spec.

   One detail: `id_a.isdigit()` might return True for strings like `'²'` (superscript), but SemVer is ASCII. I'll use `id_a.isdecimal()` or just check `id_a.isdigit()`. It's fine. To be strictly ASCII, `id_a[0].isdigit()` isn't enough. I'll stick with `id_a.isdigit()`. Actually, a safer check for SemVer numeric identifier is `id_a.isnumeric()` or `id_a.isdigit()`. I'll use `id_a.isdigit()`.

   Wait, what about identifiers that contain only digits but represent numbers? `isdigit()` is fine.

   Let's consider `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
   `pre_a = ['alpha', '1']`, `pre_b = ['alpha', 'beta']`
   i=0: alpha vs alpha -> equal
   i=1: '1' vs 'beta' -> is_num_a=True, is_num_b=False -> returns -1. Correct.

   Consider `1.0.0-1` vs `1.0.0-1.0`
   i=0: '1' vs '1' -> equal
   loop ends. len_a=1, len_b=2 -> returns -1. Correct.

   Consider `1.0.0-0` vs `1.0.0-1`
   i=0: 0 vs 1 -> returns -1. Correct.

   All good.

   I will structure the final answer with the function and the tests. I'll use `unittest` or simple assertions. I'll provide a clean script.

   One minor point: The prompt says "Return -1 if a < b, 0 if equal, 1 if a > b."
   I'll make sure the function signature matches exactly.

   Code structure:
   ```python
   def compare_versions(a: str, b: str) -> int:
       def parse(v: str):
           v = v.split('+')[0]
           if '-' in v:
               core, pre = v.split('-', 1)
               pre_parts = pre.split('.')
           else:
               core, pre_parts = v, None
           major, minor, patch = map(int, core.split('.'))
           return major, minor, patch, pre_parts

       maj_a, min_a, pat_a, pre_a = parse(a)
       maj_b, min_b, pat_b, pre_b = parse(b)

       if (maj_a, min_a, pat_a) != (maj_b, min_b, pat_b):
           return 1 if (maj_a, min_a, pat_a) > (maj_b, min_b, pat_b) else -1

       if pre_a is None and pre_b is None:
           return 0
       if pre_a is not None and pre_b is None:
           return -1
       if pre_a is None and pre_b is not None:
           return 1

       for i in range(min(len(pre_a), len(pre_b))):
           id_a, id_b = pre_a[i], pre_b[i]
           is_num_a = id_a.isdigit()
           is_num_b = id_b.isdigit()

           if is_num_a and is_num_b:
               va, vb = int(id_a), int(id_b)
               if va != vb:
                   return 1 if va > vb else -1
           elif is_num_a != is_num_b:
               return -1 if is_num_a else 1
           else:
               if id_a != id_b:
                   return 1 if id_a > id_b else -1

       if len(pre_a) != len(pre_b):
           return 1 if len(pre_a) > len(pre_b) else -1

       return 0
   ```

   Wait, `id_a.isdigit()` might behave unexpectedly with some unicode, but for SemVer it's fine. To be perfectly safe with ASCII, I could use `id_a.isdecimal()` or `id_a.isdigit()`. I'll stick with `isdigit()`. Actually, SemVer spec says numeric identifiers consist of digits. `isdigit()` is standard.

   Let's test mentally: `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")`
   `pre_a = ['alpha', '1']`, `pre_b = ['alpha', 'beta']`
   i=0: alpha, alpha -> equal
   i=1: '1', 'beta' -> is_num_a=True, is_num_b=False -> returns -1. Correct.

   One thing: `id_a.isdigit()` returns True for `'1'`, False for `'beta'`. Correct.
   What about `'01'`? `isdigit()` is True. `int('01')` is 1. Correct.

   All good.

   Now, the tests. I'll write them as a simple block that can be run.
   I'll make sure to cover the exact cases requested.
   1. pre-release vs release
   2. numeric vs alphanumeric identifiers
   3. differing field counts
   4. build metadata ignored
