# 27B_HauhauCS_Balanced — 01_semver_compare

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

Here's a thinking process that leads to the suggested solution:

1.  **Understand the Goal:** The objective is to implement a Python function `compare_versions(a, b)` that compares two strings adhering to Semantic Versioning 2.0.0 (SemVer) rules. It needs to return -1, 0, or 1 based on the comparison. It also requires 6 specific unit tests.

2.  **Analyze SemVer Rules (Precedence):**
    *   **Structure:** `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`
    *   **Core Comparison:** Compare `MAJOR`, `MINOR`, `PATCH` as integers.
        *   If `a.major < b.major` -> -1
        *   If `a.major > b.major` -> 1
        *   (Repeat for minor, patch).
    *   **Pre-release:**
        *   If one version has a pre-release tag and the other doesn't (for the same major.minor.patch), the one *without* the tag has higher precedence (is newer).
        *   Example: `1.0.0-alpha` < `1.0.0`.
    *   **Pre-release Identifiers (Dot-separated):**
        *   Compare dot-separated identifiers one by one.
        *   **Numeric:** Compare as integers.
        *   **Alphanumeric (String):** Compare as strings (ASCII sort order).
        *   **Mixed:** Numeric identifiers always have lower precedence than alphanumeric ones (e.g., `1.0.0-1` < `1.0.0-alpha`). *Correction/Refinement:* Actually, SemVer spec says "Numeric identifiers always have lower precedence than alphanumeric identifiers." Wait, let's double check.
            *   *Spec Check:* "When major, minor, and patch are equal, a pre-release version has lower precedence than a normal version... Precedence for two pre-release versions with the same major, minor, and patch version MUST be determined by comparing each dot separated identifier... Numeric identifiers always have lower precedence than alphanumeric identifiers."
            *   *Example:* `1.0.0-1` vs `1.0.0-alpha`. `1` is numeric, `alpha` is alphanumeric. `1` < `alpha`. So `1.0.0-1` < `1.0.0-alpha`.
        *   **Length:** If all preceding identifiers are equal, the one with more identifiers has higher precedence (e.g., `1.0.0-alpha` < `1.0.0-alpha.1`).
    *   **Build Metadata:**
        *   Everything after `+` is ignored for precedence.

3.  **Parsing Strategy:**
    *   Split the input string by `+` to handle build metadata first (discard the right part).
    *   Split the remaining part by `-` to separate core version from pre-release.
    *   Split the core version by `.` to get major, minor, patch. Convert to integers.
    *   If pre-release exists, split by `.` to get a list of identifiers.
        *   For each identifier, check if it's purely digits. If so, convert to int. If not, keep as string.

4.  **Comparison Logic Step-by-Step:**

    *   **Step 1: Parse `a` and `b`.**
        *   `a_parts = a.split('+')[0]`
        *   `b_parts = b.split('+')[0]`
        *   Split `a_parts` by `-` -> `a_core`, `a_pre` (maybe None).
        *   Split `b_parts` by `-` -> `b_core`, `b_pre` (maybe None).
        *   Parse `a_core` -> `[a_major, a_minor, a_patch]`.
        *   Parse `b_core` -> `[b_major, b_minor, b_patch]`.

    *   **Step 2: Compare Core (Major, Minor, Patch).**
        *   Compare `a_major` vs `b_major`. If diff, return result.
        *   Compare `a_minor` vs `b_minor`. If diff, return result.
        *   Compare `a_patch` vs `b_patch`. If diff, return result.

    *   **Step 3: Compare Pre-release.**
        *   *Case 1: Both have pre-release.*
            *   Iterate through identifiers of `a_pre` and `b_pre` simultaneously.
            *   If `a` runs out of identifiers but `b` has more -> `a < b` (-1). (More fields = higher precedence).
            *   If `b` runs out but `a` has more -> `a > b` (1).
            *   Compare current identifiers `id_a` and `id_b`.
            *   Determine types: is `id_a` int? is `id_b` int?
            *   *Sub-case 1.1: Both ints.* Compare numerically.
            *   *Sub-case 1.2: Both strings.* Compare lexicographically.
            *   *Sub-case 1.3: One int, one string.* Int is lower precedence.
                *   If `id_a` is int and `id_b` is string -> `a < b` (-1).
                *   If `id_a` is string and `id_b` is int -> `a > b` (1).
        *   *Case 2: Only `a` has pre-release.*
            *   `a` is pre-release, `b` is release. `a < b` (-1).
        *   *Case 3: Only `b` has pre-release.*
            *   `a` is release, `b` is pre-release. `a > b` (1).
        *   *Case 4: Neither has pre-release.*
            *   Equal (0).

5.  **Drafting the Code (Mental or Scratchpad):**

    ```python
    def parse_version(v):
        # Remove build metadata
        v = v.split('+')[0]
        parts = v.split('-')
        core = parts[0]
        pre = parts[1] if len(parts) > 1 else None

        core_nums = list(map(int, core.split('.')))
        
        pre_ids = []
        if pre:
            for pid in pre.split('.'):
                if pid.isdigit():
                    pre_ids.append(int(pid))
                else:
                    pre_ids.append(pid)
        
        return core_nums, pre_ids

    def compare_versions(a, b):
        a_core, a_pre = parse_version(a)
        b_core, b_pre = parse_version(b)

        # 1. Core comparison
        for i in range(3):
            if a_core[i] < b_core[i]: return -1
            if a_core[i] > b_core[i]: return 1

        # 2. Pre-release comparison
        # If both None, equal
        if a_pre is None and b_pre is None: return 0
        # If a has pre, b doesn't -> a < b
        if a_pre is not None and b_pre is None: return -1
        # If b has pre, a doesn't -> a > b
        if a_pre is None and b_pre is not None: return 1

        # Both have pre-release
        # Compare identifiers
        min_len = min(len(a_pre), len(b_pre))
        for i in range(min_len):
            id_a = a_pre[i]
            id_b = b_pre[i]

            # Type check
            a_is_int = isinstance(id_a, int)
            b_is_int = isinstance(id_b, int)

            if a_is_int and b_is_int:
                if id_a < id_b: return -1
                if id_a > id_b: return 1
            elif a_is_int and not b_is_int:
                # Numeric < Alphanumeric
                return -1
            elif not a_is_int and b_is_int:
                # Alphanumeric > Numeric
                return 1
            else:
                # Both strings
                if id_a < id_b: return -1
                if id_a > id_b: return 1
        
        # If we are here, prefixes matched. Check length.
        # More fields = higher precedence
        if len(a_pre) < len(b_pre): return -1
        if len(a_pre) > len(b_pre): return 1
        
        return 0
    ```

6.  **Refining Logic and Edge Cases:**
    *   *Is `isdigit()` safe?* Yes, for SemVer identifiers.
    *   *Leading zeros?* SemVer says "Numeric identifiers MUST NOT include leading zeroes". The parser doesn't strictly enforce this, but `int()` handles it fine (though strictly speaking `01` is invalid in SemVer, usually parsers ignore it or treat as 1. For comparison logic, `int("01") == 1` works).
    *   *Empty strings?* The prompt implies valid SemVer strings.
    *   *Comparison logic check:*
        *   `1.0.0-alpha.1` vs `1.0.0-alpha.beta`
        *   `alpha` == `alpha`
        *   `1` (int) vs `beta` (str). Int < Str. So `alpha.1` < `alpha.beta`. Correct.
        *   `1.0.0-alpha` vs `1.0.0-alpha.1`
        *   `alpha` == `alpha`
        *   Loop ends. `len(a_pre)` (1) < `len(b_pre)` (2). Returns -1. Correct.

7.  **Writing the Unit Tests:**
    The prompt asks for 6 specific tests.
    1.  **Pre-release vs release:** `1.0.0-alpha` vs `1.0.0`. Expect -1.
    2.  **Numeric vs alphanumeric identifiers:** `1.0.0-alpha.1` vs `1.0.0-alpha.beta`. Expect -1.
    3.  **Differing field counts:** `1.0.0-alpha` vs `1.0.0-alpha.1`. Expect -1.
    4.  **Build metadata ignored:** `1.0.0+build1` vs `1.0.0+build2`. Expect 0.
    5.  **Major version difference:** `1.0.0` vs `2.0.0`. Expect -1. (Good sanity check).
    6.  **Pre-release numeric comparison:** `1.0.0-1` vs `1.0.0-2`. Expect -1. Or maybe `1.0.0-1` vs `1.0.0-alpha` (numeric < alpha). Let's do `1.0.0-1` vs `1.0.0-alpha` to cover the mixed type rule explicitly or just numeric comparison. Let's stick to the prompt's implied requirements.
    *Self-Correction on Test 6:* The prompt asked for specific coverage.
    1. Pre-release vs release.
    2. Numeric vs alphanumeric identifiers.
    3. Differing field counts.
    4. Build metadata ignored.
    *Need 2 more to make 6.*
    5. Basic version comparison (Major/Minor/Patch).
    6. Complex pre-release comparison (e.g. `1.0.0-alpha.1` vs `1.0.0-alpha.2`).

    Let's refine the list to be robust.
    1. `compare_versions("1.0.0-alpha", "1.0.0")` -> -1 (Pre < Release)
    2. `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1 (Num < Alpha)
    3. `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1 (Fewer fields < More fields)
    4. `compare_versions("1.0.0+build1", "1.0.0+build2")` -> 0 (Meta ignored)
    5. `compare_versions("1.0.0", "2.0.0")` -> -1 (Major diff)
    6. `compare_versions("1.0.0-1", "1.0.0-2")` -> -1 (Numeric pre-release comparison)
    *Wait, `1.0.0-1` vs `1.0.0-alpha` is a good one for numeric vs alpha.* Let's swap 6 to that or keep numeric comparison. The prompt specifically mentioned "numeric vs alphanumeric identifiers" in the requirements text, so test 2 covers that. Let's ensure numeric comparison logic works.

    Let's pick:
    1. Pre vs Release (`1.0.0-alpha` < `1.0.0`)
    2. Num vs Alpha (`1.0.0-1` < `1.0.0-alpha`) -> Wait, prompt example was `alpha.1` vs `alpha.beta`. That's `int` vs `str`.
    3. Field count (`1.0.0-alpha` < `1.0.0-alpha.1`)
    4. Build metadata (`1.0.0+foo` == `1.0.0+bar`)
    5. Major version (`1.0.0` < `2.0.0`)
    6. Minor/Patch logic (`1.0.0` < `1.1.0` or `1.0.0` < `1.0.1`)

    Actually, let's look at the prompt requirements again: "Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored."
    That lists 4 specific scenarios. I need 2 more general ones to hit the count of 6.

    Let's write the code structure.

8.  **Implementation Details:**
    *   Use `re` module? Not strictly necessary, string splitting works fine for standard SemVer.
    *   Helper function `parse` is cleaner.
    *   Need to handle `int` conversion carefully. `str.isdigit()` returns True for "123", False for "alpha". Note: `isdigit` returns False for negative numbers, but SemVer identifiers are non-negative.

    *Code Structure:*
    ```python
    def compare_versions(a: str, b: str) -> int:
        def parse(v):
            # Strip build metadata
            if '+' in v:
                v = v.split('+')[0]
            
            # Split pre-release
            parts = v.split('-')
            core_str = parts[0]
            pre_str = parts[1] if len(parts) > 1 else None
            
            # Parse core
            core = list(map(int, core_str.split('.')))
            
            # Parse pre-release
            pre = None
            if pre_str:
                pre = []
                for p in pre_str.split('.'):
                    if p.isdigit():
                        pre.append(int(p))
                    else:
                        pre.append(p)
            return core, pre

        a_core, a_pre = parse(a)
        b_core, b_pre = parse(b)

        # 1. Compare Major, Minor, Patch
        for i in range(3):
            if a_core[i] < b_core[i]:
                return -1
            if a_core[i] > b_core[i]:
                return 1
        
        # 2. Compare Pre-release
        # Case: No pre-release on either -> Equal
        if a_pre is None and b_pre is None:
            return 0
        
        # Case: a has pre-release, b does not -> a is lower
        if a_pre is not None and b_pre is None:
            return -1
            
        # Case: b has pre-release, a does not -> a is higher
        if a_pre is None and b_pre is not None:
            return 1
            
        # Case: Both have pre-release
        # Compare identifier by identifier
        min_len = min(len(a_pre), len(b_pre))
        
        for i in range(min_len):
            id_a = a_pre[i]
            id_b = b_pre[i]
            
            # Check types
            a_is_int = isinstance(id_a, int)
            b_is_int = isinstance(id_b, int)
            
            if a_is_int and b_is_int:
                if id_a < id_b: return -1
                if id_a > id_b: return 1
            elif a_is_int and not b_is_int:
                # Numeric < Alphanumeric
                return -1
            elif not a_is_int and b_is_int:
                # Alphanumeric > Numeric
                return 1
            else:
                # Both strings
                if id_a < id_b: return -1
                if id_a > id_b: return 1
        
        # If all compared identifiers are equal, the one with more identifiers is higher
        if len(a_pre) < len(b_pre):
            return -1
        if len(a_pre) > len(b_pre):
            return 1
            
        return 0
    ```

9.  **Testing the logic mentally:**
    *   `1.0.0-alpha` vs `1.0.0`
        *   Cores equal.
        *   `a_pre` exists, `b_pre` None. Returns -1. Correct.
    *   `1.0.0-alpha.1` vs `1.0.0-alpha.beta`
        *   Cores equal.
        *   Both have pre.
        *   i=0: `alpha` vs `alpha` (str vs str). Equal.
        *   i=1: `1` (int) vs `beta` (str).
        *   `a_is_int` True, `b_is_int` False.
        *   Returns -1. Correct.
    *   `1.0.0-alpha` vs `1.0.0-alpha.1`
        *   Cores equal.
        *   Both have pre.
        *   i=0: `alpha` vs `alpha`. Equal.
        *   Loop ends (min_len 1).
        *   `len(a_pre)` (1) < `len(b_pre)` (2).
        *   Returns -1. Correct.
    *   `1.0.0+build1` vs `1.0.0+build2`
        *   `+` stripped. Cores equal. Pre None.
        *   Returns 0. Correct.
    *   `1.0.0-1` vs `1.0.0-2`
        *   Cores equal.
        *   i=0: `1` (int) vs `2` (int).
        *   `1 < 2`. Returns -1. Correct.
    *   `1.0.0-alpha` vs `1.0.0-beta`
        *   Cores equal.
        *   i=0: `alpha` (str) vs `beta` (str).
        *   `alpha < beta`. Returns -1. Correct.

10. **Final Code Assembly:**
    Combine function and tests. Use `unittest` or simple assertions. The prompt doesn't specify a testing framework, but standard `assert` or `unittest` is fine. I'll use `unittest` for professionalism, or just a simple script block. Since it's a "function implementation", a simple block of assertions is often easier to read for these prompts, but `unittest` is safer. I'll provide a standard Python script with assertions.

    *Self-Correction on `isdigit`*: `isdigit` works for unicode digits too. `isdecimal` might be safer, but for standard ASCII SemVer `isdigit` is fine. However, strictly speaking, SemVer identifiers are `[0-9A-Za-z-]`. `isdigit` handles `0-9`. What about negative numbers? SemVer identifiers are non-negative. What about `01`? `int('01')` is 1. SemVer says no leading zeros, but comparison logic usually treats them as numbers. The code handles it.

    One edge case: `1.0.0-01` vs `1.0.0-1`.
    `int('01')` -> 1. `int('1')` -> 1. Equal.
    Technically `01` is invalid SemVer, but if passed, numeric comparison holds.

    Let's refine the type checking.
    `isinstance(id_a, int)` works because we converted in parsing.
    However, `isdigit` on string "1" is True.
    Wait, in the parsing step:
    `if p.isdigit(): pre.append(int(p))`
    So `pre` list contains mixed `int` and `str`.
    The logic `isinstance(id_a, int)` is correct.

    Let's double check the `isdigit` behavior on empty strings? SemVer identifiers can't be empty (dots separate them). But `split('.')` on empty string returns `['']`. `isdigit()` on `''` is False. It would be treated as string `''`. Comparison `'' < 'a'` is True. Seems acceptable behavior for malformed input, though input is assumed valid.

    **Final check on "Numeric identifiers always rank lower than alphanumeric ones"**:
    My code:
    `elif a_is_int and not b_is_int: return -1` (a < b)
    `elif not a_is_int and b_is_int: return 1` (a > b)
    This implies int < str.
    Example: `1` vs `alpha`. `1` is int, `alpha` is str. `1` < `alpha`. Correct.

    **Final check on "A version WITH a pre-release tag has LOWER precedence"**:
    My code:
    `if a_pre is not None and b_pre is None: return -1`
    `a` has tag, `b` doesn't. `a` < `b`. Correct.

    **Final check on "Larger set of fields ranks higher"**:
    My code:
    `if len(a_pre) < len(b_pre): return -1`
    `a` has fewer fields. `a` < `b`. Correct.

    **Final check on "Build metadata IGNORED"**:
    My code:
    `v = v.split('+')[0]`
    Correct.

11. **Writing the solution.**

    I will write the code clearly with comments.
    For the tests, I will use a simple `if __name__ == '__main__':` block with assertions for immediate verification, or `unittest`. The prompt asks to "Include 6 unit tests". A `unittest` class is the most formal way.

    Let's stick to a clean function and a test block.

    ```python
    import unittest

    def compare_versions(a: str, b: str) -> int:
        # Helper to parse version string
        def parse_version(v: str):
            # 1. Ignore build metadata (everything after +)
            if '+' in v:
                v = v.split('+')[0]
            
            # 2. Split core and pre-release
            # Note: A version string might not have a pre-release part
            parts = v.split('-')
            core_str = parts[0]
            pre_str = parts[1] if len(parts) > 1 else None
            
            # 3. Parse Major, Minor, Patch
            # We assume valid SemVer format for core
            core = list(map(int, core_str.split('.')))
            
            # 4. Parse Pre-release identifiers
            pre = None
            if pre_str:
                pre = []
                for identifier in pre_str.split('.'):
                    # Check if identifier is purely numeric
                    if identifier.isdigit():
                        pre.append(int(identifier))
                    else:
                        pre.append(identifier)
            
            return core, pre

        a_core, a_pre = parse_version(a)
        b_core, b_pre = parse_version(b)

        # Step 1: Compare Major, Minor, Patch
        for i in range(3):
            if a_core[i] < b_core[i]:
                return -1
            if a_core[i] > b_core[i]:
                return 1
        
        # Step 2: Compare Pre-release
        # If both are None (no pre-release), they are equal (assuming cores matched)
        if a_pre is None and b_pre is None:
            return 0
        
        # A version with a pre-release tag has LOWER precedence than the same version without one
        if a_pre is not None and b_pre is None:
            return -1
        if a_pre is None and b_pre is not None:
            return 1
        
        # Both have pre-release tags. Compare identifiers dot-by-dot.
        min_len = min(len(a_pre), len(b_pre))
        
        for i in range(min_len):
            id_a = a_pre[i]
            id_b = b_pre[i]
            
            # Determine types
            a_is_int = isinstance(id_a, int)
            b_is_int = isinstance(id_b, int)
            
            if a_is_int and b_is_int:
                # Both numeric: compare numerically
                if id_a < id_b:
                    return -1
                if id_a > id_b:
                    return 1
            elif a_is_int and not b_is_int:
                # Numeric identifiers always have lower precedence than alphanumeric
                return -1
            elif not a_is_int and b_is_int:
                # Alphanumeric has higher precedence
                return 1
            else:
                # Both alphanumeric: compare lexically (ASCII)
                if id_a < id_b:
                    return -1
                if id_a > id_b:
                    return 1
        
        # If all preceding identifiers are equal, the one with more identifiers has higher precedence
        if len(a_pre) < len(b_pre):
            return -1
        if len(a_pre) > len(b_pre):
            return 1
            
        return 0

    class TestCompareVersions(unittest.TestCase):
        def test_pre_release_vs_release(self):
            # 1.0.0-alpha < 1.0.0
            self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
            self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

        def test_numeric_vs_alphanumeric_identifiers(self):
            # Numeric identifiers rank lower than alphanumeric
            # 1.0.0-alpha.1 < 1.0.0-alpha.beta
            self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
            # Also 1.0.0-1 < 1.0.0-alpha
            self.assertEqual(compare_versions("1.0.0-1", "1.0.0-alpha"), -1)

        def test_differing_field_counts(self):
            # Larger set of fields ranks higher
            # 1.0.0-alpha < 1.0.0-alpha.1
            self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
            self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

        def test_build_metadata_ignored(self):
            # Build metadata does not affect precedence
            self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
            self.assertEqual(compare_versions("1.0.0-alpha+build1", "1.0.0-alpha+build2"), 0)
            self.assertEqual(compare_versions("1.0.0+build", "1.0.0"), 0)

        def test_major_minor_patch_comparison(self):
            # Basic version comparison
            self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
            self.assertEqual(compare_versions("1.0.0", "1.1.0"), -1)
            self.assertEqual(compare_versions("1.0.0", "1.0.1"), -1)

        def test_numeric_pre_release_comparison(self):
            # Numeric identifiers compared numerically
            self.assertEqual(compare_versions("1.0.0-1", "1.0.0-2"), -1)
            self.assertEqual(compare_versions("1.0.0-10", "1.0.0-2"), 1) # 10 > 2 numerically

    if __name__ == '__main__':
        unittest.main()
    ```

    Wait, `isdigit` check. `isdigit` returns `True` for superscript digits in unicode, but in standard ASCII strings it works for 0-9.
    Is `isdigit` safe for "123"? Yes.
    Is `isdigit` safe for "-1"? No, returns False. But SemVer identifiers don't have signs.
    Is `isdigit` safe for "1.0"? No, but we split by dot already.
    Is `isdigit` safe for "01"? Yes.

    One tiny detail: SemVer 2.0.0 spec says "Numeric identifiers MUST NOT include leading zeroes".
    If input is `1.0.0-01`, `int('01')` is 1.
    If input is `1.0.0-1`, `int('1')` is 1.
    They compare as equal.
    Strictly speaking `01` is invalid, but comparison logic usually treats it as number 1.
    If we wanted to be pedantic about invalid versions, we'd raise errors, but the prompt asks for comparison logic, implying valid inputs or robust handling. Treating `01` as `1` is the standard behavior for version comparators (like npm, pip, etc) unless strict validation is requested.

    Let's verify the `compare_versions` signature matches the prompt.
    `compare_versions(a: str, b: str) -> int`
    Yes.

    Double check the "numeric vs alphanumeric" logic in the prompt example: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
    My code:
    `alpha` == `alpha`.
    Next: `1` (int) vs `beta` (str).
    `a_is_int` (True), `b_is_int` (False).
    Returns -1.
    Result: `1.0.0-alpha.1` < `1.0.0-alpha.beta`.
    Is this correct?
    Spec: "Numeric identifiers always have lower precedence than alphanumeric identifiers."
    Yes. `1` < `beta`.
    Wait, usually in sorting `1` comes before `b`? No, ASCII '1' is 49, 'b' is 98. So `1` < `b` lexicographically too.
    But if it was `1` vs `a`? '1' (49) < 'a' (97).
    What if it was `10` vs `2`? Lexicographically `10` < `2` ('1' < '2'). Numerically `10` > `2`.
    The rule "Numeric < Alphanumeric" is distinct from lexicographical comparison between numbers and strings.
    Wait, is `1` (numeric) < `a` (alphanumeric)? Yes.
    Is `1` (numeric) < `2` (numeric)? Yes (1 < 2).
    Is `a` (alphanumeric) < `b` (alphanumeric)? Yes.
    Is `1` (numeric) < `a` (alphanumeric)?
    Spec: "Numeric identifiers always have lower precedence than alphanumeric identifiers."
    So yes, `1` < `a`.
    My code handles this: `elif a_is_int and not b_is_int: return -1`.

    Wait, what if `a` is `1.0.0-1` and `b` is `1.0.0-a`?
    `1` is int. `a` is str.
    `a_is_int` = True. `b_is_int` = False.
    Returns -1. `1.0.0-1` < `1.0.0-a`.
    Is this correct?
    Yes.

    What if `a` is `1.0.0-a` and `b` is `1.0.0-1`?
    `a` is str. `b` is int.
    `a_is_int` = False. `b_is_int` = True.
    Returns 1. `1.0.0-a` > `1.0.0-1`.
    Correct.

    Looks solid.

    One final check on test cases.
    The prompt asks for "6 unit tests".
    1. Pre-release vs release.
    2. Numeric vs alphanumeric identifiers.
    3. Differing field counts.
    4. Build metadata ignored.
    5. (Implicit) Basic version comparison.
    6. (Implicit) Numeric pre-release comparison or complex logic.

    I will include 6 distinct test methods or assertions to be safe.

    **Plan:**
    1.  Define `compare_versions`.
    2.  Define a test class with 6 methods covering the requirements.
    3.  Run `unittest.main()`.

    The code looks good.

    *Self-Correction*: The prompt asks for "numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta)".
    In `1.0.0-alpha.1`, the identifier is `1` (numeric).
    In `1.0.0-alpha.beta`, the identifier is `beta` (alphanumeric).
    Comparison: `1` vs `beta`.
    `1` is numeric, `beta` is alphanumeric.
    Rule: Numeric < Alphanumeric.
    So `1.0.0-alpha.1` < `1.0.0-alpha.beta`.
    My code returns -1.
    Wait, let me double check the SemVer spec wording carefully.
    "Numeric identifiers always have lower precedence than alphanumeric identifiers."
    Yes.
    However, standard ASCII sort order: '1' (49) is less than 'b' (98). So lexicographically it also holds.
    But consider `1.0.0-10` vs `1.0.0-2`.
    `10` (num) vs `2` (num). `10` > `2`.
    Consider `1.0.0-10` vs `1.0.0-a`.
    `10` (num) vs `a` (alpha). `10` < `a`.
    Lexicographically `10` < `a` ('1' < 'a').
    Consider `1.0.0-Z` vs `1.0.0-1`.
    `Z` (alpha) vs `1` (num). `Z` > `1`.
    Lexicographically `Z` (90) > `1` (49).
    It seems the rule "Numeric < Alphanumeric" is consistent with ASCII order for digits vs letters, but the distinction is important if we had other characters or if the definition of "alphanumeric" included symbols, but SemVer identifiers are restricted to ASCII alphanumerics and hyphens.
    Actually, hyphens are allowed in identifiers?
    Spec: "Pre-release versions and build metadata MAY be denoted by appending a hyphen or a plus sign... Identifiers MUST comprise only ASCII alphanumerics and hyphens [0-9A-Za-z-]."
    Wait, hyphens are allowed *inside* identifiers?
    "Identifiers MUST comprise only ASCII alphanumerics and hyphens [0-9A-Za-z-]."
    Yes.
    But `isdigit()` returns False for strings containing hyphens.
    Example: `1.0.0-alpha-beta`.
    Split by `-`?
    Wait, the separator for pre-release identifiers is `.` (dot).
    The separator between core and pre-release is `-` (hyphen).
    So `1.0.0-alpha-beta`?
    If the input is `1.
