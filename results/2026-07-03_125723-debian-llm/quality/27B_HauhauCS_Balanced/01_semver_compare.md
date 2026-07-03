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

Here's a thinking process that leads to the suggested implementation:

1.  **Understand the Goal:** The objective is to write a Python function `compare_versions(a, b)` that compares two SemVer 2.0.0 strings according to the specific precedence rules defined in the specification. It also needs to include 6 specific unit tests.

2.  **Analyze SemVer 2.0.0 Precedence Rules:**
    *   **Structure:** `Major.Minor.Patch[-Prerelease][+Build]`
    *   **Basic Comparison:** Compare Major, then Minor, then Patch. Higher number wins.
    *   **Pre-release vs. Release:** `1.0.0-alpha` < `1.0.0`. A version *without* a pre-release tag has higher precedence than one *with* a tag for the same Major.Minor.Patch.
    *   **Pre-release Identifiers:** Separated by dots (e.g., `alpha.1.beta`).
        *   Compare dot by dot.
        *   **Numeric vs. Alphanumeric:** Numeric identifiers are compared as integers. Alphanumeric (containing non-digits) are compared lexically (ASCII).
        *   **Type Precedence:** Numeric identifiers always have lower precedence than alphanumeric identifiers (e.g., `1` < `alpha`).
        *   **Length:** If all preceding identifiers are equal, a larger set of fields (more dots) has higher precedence (e.g., `1.0.0-alpha` < `1.0.0-alpha.1`).
    *   **Build Metadata:** Everything after `+` is ignored.

3.  **Algorithm Design:**

    *   *Step 1: Parsing.* Split the input string into core version (Major.Minor.Patch), pre-release, and build metadata.
        *   Regex is useful here, or simple string splitting.
        *   Pattern: `^(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z\-]+(?:\.[0-9A-Za-z\-]+)*))?(?:\+([0-9A-Za-z\-]+(?:\.[0-9A-Za-z\-]+)*))?$` (Though strict validation isn't explicitly requested, robust parsing is needed). Let's stick to simple splitting logic for the comparison logic to keep it readable, assuming valid inputs as per standard function contracts, but handling the specific separators `-` and `+`.

    *   *Step 2: Compare Core Version (Major, Minor, Patch).*
        *   Split by `.`. Convert to int.
        *   Compare A.Major vs B.Major. If different, return sign.
        *   Compare A.Minor vs B.Minor. If different, return sign.
        *   Compare A.Patch vs B.Patch. If different, return sign.
        *   If equal, proceed to Pre-release.

    *   *Step 3: Compare Pre-release.*
        *   Let $P_a$ be pre-release of A, $P_b$ be pre-release of B.
        *   **Case 1:** $P_a$ exists, $P_b$ does not. -> A < B (Return -1).
        *   **Case 2:** $P_b$ exists, $P_a$ does not. -> A > B (Return 1).
        *   **Case 3:** Both exist. Split both by `.` into lists of identifiers.
        *   Iterate through indices $i$ from 0 to min(len($P_a$), len($P_b$)).
            *   Let $id_a = P_a[i]$, $id_b = P_b[i]$.
            *   Determine if $id_a$ is numeric (all digits) or alphanumeric. Same for $id_b$.
            *   **Comparison Logic:**
                *   If both numeric: compare as integers.
                *   If both alphanumeric: compare strings (ASCII).
                *   If one numeric, one alphanumeric: Numeric < Alphanumeric.
            *   If $id_a < id_b$, return -1. If $id_a > id_b$, return 1.
        *   **Case 4:** All compared identifiers are equal.
            *   If len($P_a$) < len($P_b$), A < B (Return -1).
            *   If len($P_a$) > len($P_b$), A > B (Return 1).
            *   If lengths equal, versions are equal regarding pre-release.

    *   *Step 4: Build Metadata.*
        *   Ignored.

    *   *Step 5: Final Result.*
        *   If all checks passed and no difference found, return 0.

4.  **Drafting the Code (Mental or Scratchpad):**

    ```python
    def compare_versions(a, b):
        # 1. Strip build metadata
        # Split at '+'
        a_core = a.split('+')[0]
        b_core = b.split('+')[0]

        # 2. Split into version and pre-release
        # Split at '-'
        # Note: Pre-release can contain '-', so splitting on first '-' is safer?
        # Actually SemVer spec says pre-release is a series of dot-separated identifiers.
        # The first '-' after the patch number starts the pre-release.
        # However, standard split('-') might be risky if not careful, but usually
        # version numbers don't have '-' inside them.
        # Let's use regex or find index of first '-' after patch.
        # Simpler: split by '-', take first part as version, rest joined by '-' is pre-release?
        # Wait, 1.0.0-alpha-1. The pre-release is "alpha-1"? No, identifiers are dot separated.
        # 1.0.0-alpha.1.
        # The separator between patch and pre-release is '-'.
        # The separator between pre-release identifiers is '.'.
        # So split string by '-'. First part is version.
        # If there is a second part, it's pre-release.
        # BUT, build metadata comes after '+'.
        # So: 1.0.0-alpha+build -> split('+') -> ['1.0.0-alpha', 'build']
        # Then split('1.0.0-alpha', '-') -> ['1.0.0', 'alpha']
        
        # Let's refine parsing.
        def parse(ver_str):
            # Remove build metadata
            if '+' in ver_str:
                ver_str = ver_str.split('+')[0]
            
            # Split pre-release
            if '-' in ver_str:
                parts = ver_str.split('-', 1) # Split only once
                version_part = parts[0]
                pre_release = parts[1]
            else:
                version_part = ver_str
                pre_release = None
            
            # Parse major.minor.patch
            v_parts = version_part.split('.')
            major = int(v_parts[0])
            minor = int(v_parts[1])
            patch = int(v_parts[2])
            
            return major, minor, patch, pre_release

        # ... comparison logic ...
    ```

    *Refining Pre-release Comparison Logic:*
    Identifiers are dot separated.
    `1.0.0-alpha.1` -> identifiers `['alpha', '1']`.
    `1.0.0-alpha.beta` -> identifiers `['alpha', 'beta']`.

    Comparison of `id1` vs `id2`:
    1. Check if numeric. `isdigit()` works for pure numbers.
    2. If `id1` numeric and `id2` numeric: `int(id1)` vs `int(id2)`.
    3. If `id1` alphanumeric and `id2` alphanumeric: `id1` vs `id2` (string compare).
    4. If mixed: numeric < alphanumeric.

    *Edge Case Check:*
    `1.0.0-1` vs `1.0.0-alpha`.
    `1` is numeric. `alpha` is alphanumeric.
    Rule: "numeric identifiers always have lower precedence than alphanumeric identifiers".
    So `1` < `alpha`. Correct.

    *Edge Case Check:*
    `1.0.0-alpha` vs `1.0.0-alpha.1`.
    Lists: `['alpha']` vs `['alpha', '1']`.
    Loop runs for index 0. `alpha` == `alpha`.
    Loop ends.
    Length check: len 1 < len 2.
    So `1.0.0-alpha` < `1.0.0-alpha.1`. Correct.

5.  **Writing the Implementation:**

    ```python
    def compare_versions(a: str, b: str) -> int:
        # Helper to parse version string
        def parse_version(v_str):
            # Handle build metadata (ignore it)
            if '+' in v_str:
                v_str = v_str.split('+')[0]
            
            # Handle pre-release
            pre_release = None
            if '-' in v_str:
                # Split at the first hyphen. 
                # Note: Hyphens are allowed in identifiers, but the separator 
                # between patch and pre-release is a hyphen.
                # Spec: "Pre-release version is denoted by appending a hyphen..."
                parts = v_str.split('-', 1)
                core = parts[0]
                pre_release = parts[1]
            else:
                core = v_str
            
            # Parse Major.Minor.Patch
            # Assuming valid SemVer format as per prompt context
            core_parts = core.split('.')
            major = int(core_parts[0])
            minor = int(core_parts[1])
            patch = int(core_parts[2])
            
            return major, minor, patch, pre_release

        maj_a, min_a, pat_a, pre_a = parse_version(a)
        maj_b, min_b, pat_b, pre_b = parse_version(b)

        # 1. Compare Major
        if maj_a != maj_b:
            return 1 if maj_a > maj_b else -1
        # 2. Compare Minor
        if min_a != min_b:
            return 1 if min_a > min_b else -1
        # 3. Compare Patch
        if pat_a != pat_b:
            return 1 if pat_a > pat_b else -1

        # 4. Compare Pre-release
        # If both are None (no pre-release), they are equal (so far)
        if pre_a is None and pre_b is None:
            return 0
        
        # If a has pre-release and b doesn't, a < b
        if pre_a is not None and pre_b is None:
            return -1
        
        # If b has pre-release and a doesn't, a > b
        if pre_a is None and pre_b is not None:
            return 1
        
        # Both have pre-release, compare identifiers
        # Split by dot
        ids_a = pre_a.split('.')
        ids_b = pre_b.split('.')

        # Compare dot by dot
        min_len = min(len(ids_a), len(ids_b))
        for i in range(min_len):
            id_a = ids_a[i]
            id_b = ids_b[i]
            
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
                # Both alphanumeric (string comparison)
                if id_a != id_b:
                    # Lexical comparison (ASCII)
                    # Python string comparison is ASCII compatible
                    return 1 if id_a > id_b else -1
        
        # If all compared identifiers are equal, compare lengths
        if len(ids_a) != len(ids_b):
            return 1 if len(ids_a) > len(ids_b) else -1
            
        return 0
    ```

6.  **Developing Unit Tests:**
    The prompt requires 6 specific tests covering:
    1.  Pre-release vs release.
    2.  Numeric vs alphanumeric identifiers.
    3.  Differing field counts.
    4.  Build metadata ignored.
    5.  (Implicit) Standard version comparison.
    6.  (Implicit) Complex pre-release comparison.

    Let's map them out:
    1.  `1.0.0-alpha` vs `1.0.0` -> -1 (Pre-release < Release).
    2.  `1.0.0-alpha.1` vs `1.0.0-alpha.beta` -> -1 (Numeric '1' < Alphanumeric 'beta').
    3.  `1.0.0-alpha` vs `1.0.0-alpha.1` -> -1 (Fewer fields < More fields).
    4.  `1.0.0+build.1` vs `1.0.0+build.2` -> 0 (Build metadata ignored).
    5.  `1.0.0` vs `2.0.0` -> -1 (Major version).
    6.  `1.0.0-alpha.1` vs `1.0.0-alpha.2` -> -1 (Numeric comparison).

    Wait, the prompt asked for specific coverage. Let's ensure the tests match the description.
    - "pre-release vs release" -> Test 1.
    - "numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta)" -> Test 2.
    - "differing field counts (1.0.0-alpha vs 1.0.0-alpha.1)" -> Test 3.
    - "build metadata is ignored" -> Test 4.
    I need 2 more to make 6. I'll add standard numeric comparison and maybe a reverse case for pre-release.

    Let's refine the test list to be robust.
    1.  `compare_versions("1.0.0-alpha", "1.0.0")` -> -1
    2.  `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1
    3.  `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1
    4.  `compare_versions("1.0.0+build.1", "1.0.0+build.2")` -> 0
    5.  `compare_versions("1.0.0", "1.0.1")` -> -1 (Basic patch check)
    6.  `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.2")` -> -1 (Numeric ID comparison)

    Wait, checking the prompt constraints again.
    "Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers..., differing field counts..., and that build metadata is ignored."
    It lists 4 specific requirements. I just need to add 2 more generic ones or split the logic. The list above covers all.

7.  **Refining the Code Logic (Self-Correction/Verification):**

    *   *String Comparison:* Python's `<` and `>` on strings use ASCII values. This matches "lexically in ASCII order".
    *   *Numeric Check:* `isdigit()` returns True for "123", False for "alpha". What about "01"? `isdigit` is True, `int("01")` is 1. SemVer says "Numeric identifiers are non-negative integers". Leading zeros are technically not allowed in strict SemVer (e.g., `01` is invalid), but usually parsers handle them or treat them as integers. The prompt doesn't specify strict validation, just precedence. `int()` handles leading zeros fine.
    *   *Hyphen in identifiers:* SemVer allows hyphens in identifiers (e.g., `1.0.0-alpha-1`).
        *   My parsing logic: `v_str.split('-', 1)`.
        *   Input: `1.0.0-alpha-1`.
        *   Split: `['1.0.0', 'alpha-1']`.
        *   Pre-release: `alpha-1`.
        *   Splitting pre-release by `.`: `['alpha-1']`.
        *   Wait, if the input is `1.0.0-alpha.1`, split by `.` gives `['alpha', '1']`.
        *   If input is `1.0.0-alpha-1`, split by `.` gives `['alpha-1']`. This is treated as one alphanumeric identifier.
        *   Is this correct? Yes. The dot is the separator for precedence fields. Hyphens are just characters within an identifier.
    *   *Parsing Edge Case:* `1.0.0-1.0.0`.
        *   Split `-` -> `['1.0.0', '1.0.0']`.
        *   Pre-release `1.0.0`.
        *   Split `.` -> `['1', '0', '0']`.
        *   Comparison works.

    *   *Parsing Edge Case:* `1.0.0` (no hyphen).
        *   `if '-' in v_str` is False.
        *   `pre_release` is None.
        *   Logic handles it.

    *   *Parsing Edge Case:* `1.0.0+build`.
        *   `if '+' in v_str` -> True.
        *   `v_str` becomes `1.0.0`.
        *   `if '-' in v_str` -> False.
        *   `pre_release` is None.
        *   Logic handles it.

    *   *Parsing Edge Case:* `1.0.0-alpha+build`.
        *   `if '+'` -> `1.0.0-alpha`.
        *   `if '-'` -> `['1.0.0', 'alpha']`.
        *   `pre_release` = `alpha`.
        *   Logic handles it.

8.  **Final Code Structure:**

    ```python
    import unittest

    def compare_versions(a: str, b: str) -> int:
        def parse(v_str):
            # 1. Strip build metadata
            if '+' in v_str:
                v_str = v_str.split('+')[0]
            
            # 2. Split pre-release
            pre_release = None
            if '-' in v_str:
                # Split at the first hyphen found after the patch version
                # Since Major.Minor.Patch doesn't contain hyphens, 
                # the first hyphen is the separator.
                parts = v_str.split('-', 1)
                core = parts[0]
                pre_release = parts[1]
            else:
                core = v_str
            
            # 3. Parse Major.Minor.Patch
            parts = core.split('.')
            major = int(parts[0])
            minor = int(parts[1])
            patch = int(parts[2])
            
            return major, minor, patch, pre_release

        maj_a, min_a, pat_a, pre_a = parse(a)
        maj_b, min_b, pat_b, pre_b = parse(b)

        # Compare Major
        if maj_a != maj_b:
            return 1 if maj_a > maj_b else -1
        # Compare Minor
        if min_a != min_b:
            return 1 if min_a > min_b else -1
        # Compare Patch
        if pat_a != pat_b:
            return 1 if pat_a > pat_b else -1

        # Compare Pre-release
        # Case 1: Both are release versions (no pre-release)
        if pre_a is None and pre_b is None:
            return 0
        
        # Case 2: a is pre-release, b is release -> a < b
        if pre_a is not None and pre_b is None:
            return -1
        
        # Case 3: a is release, b is pre-release -> a > b
        if pre_a is None and pre_b is not None:
            return 1
        
        # Case 4: Both are pre-release
        # Split identifiers by dot
        ids_a = pre_a.split('.')
        ids_b = pre_b.split('.')

        # Compare dot by dot
        length = min(len(ids_a), len(ids_b))
        for i in range(length):
            id_a = ids_a[i]
            id_b = ids_b[i]
            
            # Check if numeric
            is_num_a = id_a.isdigit()
            is_num_b = id_b.isdigit()
            
            if is_num_a and is_num_b:
                # Numeric comparison
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
                # Alphanumeric vs Alphanumeric (Lexical)
                if id_a != id_b:
                    return 1 if id_a > id_b else -1
        
        # If all compared identifiers are equal, the one with more identifiers is higher
        if len(ids_a) != len(ids_b):
            return 1 if len(ids_a) > len(ids_b) else -1
            
        return 0

    # Unit Tests
    class TestCompareVersions(unittest.TestCase):
        def test_pre_release_vs_release(self):
            # 1.0.0-alpha < 1.0.0
            self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
            self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

        def test_numeric_vs_alphanumeric_identifiers(self):
            # 1.0.0-alpha.1 < 1.0.0-alpha.beta (numeric < alphanumeric)
            self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
            self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1"), 1)

        def test_differing_field_counts(self):
            # 1.0.0-alpha < 1.0.0-alpha.1 (fewer fields < more fields)
            self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
            self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

        def test_build_metadata_ignored(self):
            # 1.0.0+build.1 == 1.0.0+build.2
            self.assertEqual(compare_versions("1.0.0+build.1", "1.0.0+build.2"), 0)
            self.assertEqual(compare_versions("1.0.0-alpha+build.1", "1.0.0-alpha+build.2"), 0)

        def test_basic_version_comparison(self):
            # Standard numeric comparison
            self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
            self.assertEqual(compare_versions("1.1.0", "1.0.0"), 1)
            self.assertEqual(compare_versions("1.0.1", "1.0.0"), 1)

        def test_mixed_pre_release_comparison(self):
            # 1.0.0-alpha.1 < 1.0.0-alpha.2
            self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.2"), -1)
            # 1.0.0-1 < 1.0.0-alpha (numeric < alphanumeric)
            self.assertEqual(compare_versions("1.0.0-1", "1.0.0-alpha"), -1)

    if __name__ == '__main__':
        unittest.main()
    ```

9.  **Reviewing Constraints & Edge Cases:**
    *   *Constraint:* "numeric identifiers always rank lower than alphanumeric ones".
        *   Code: `elif is_num_a and not is_num_b: return -1`. Correct.
    *   *Constraint:* "alphanumeric compared lexically in ASCII order".
        *   Code: `return 1 if id_a > id_b else -1`. Python string comparison is ASCII based. Correct.
    *   *Constraint:* "larger set of fields ranks higher".
        *   Code: `if len(ids_a) != len(ids_b): return 1 if len(ids_a) > len(ids_b) else -1`. Correct.
    *   *Constraint:* "A version WITH a pre-release tag has LOWER precedence".
        *   Code: `if pre_a is not None and pre_b is None: return -1`. Correct.

    *   *Potential Issue:* `isdigit()` behavior.
        *   In Python 3, `isdigit()` returns True for some unicode digits, but for standard ASCII strings (which SemVer implies), it works perfectly for "0"-"9".
        *   What about empty strings? SemVer identifiers shouldn't be empty, but if `1.0.0-..1` occurred? `split('.')` might produce empty strings. `"".isdigit()` is False. `int("")` raises error.
        *   Assuming valid SemVer strings as per standard function contracts usually implies valid input. The prompt doesn't ask for validation, just comparison.

    *   *Potential Issue:* Leading zeros in numeric identifiers.
        *   SemVer spec says: "Numeric identifiers MUST NOT include leading zeroes."
        *   However, if they exist, `int()` handles them (e.g., `int("01")` -> `1`). My code handles this gracefully.

    *   *Potential Issue:* Hyphens in pre-release identifiers.
        *   Example: `1.0.0-alpha-1`.
        *   My code splits at first `-`. `pre_release` = `alpha-1`.
        *   Splits by `.`. `ids` = `['alpha-1']`.
        *   Comparison: `alpha-1` vs `alpha`.
        *   `alpha-1` is alphanumeric. `alpha` is alphanumeric.
        *   Lexical compare: `alpha-1` vs `alpha`.
        *   '-' (ASCII 45) < 'a' (ASCII 97).
        *   So `alpha-1` < `alpha`.
        *   Is this correct?
        *   Actually, `alpha-1` is a single identifier. `alpha` is a single identifier.
        *   Lexically `alpha-1` comes before `alpha`?
        *   Wait, `alpha-1` vs `alpha`.
        *   Compare char by char. `a`==`a`, `l`==`l`... `a`==`a`.
        *   Then `alpha` ends. `alpha-1` has `-`.
        *   Shorter string comes first if prefix matches? Yes.
        *   So `alpha` < `alpha-1`.
        *   Wait, if `alpha` is shorter, it is a prefix.
        *   In ASCII sort, shorter string comes first.
        *   So `alpha` < `alpha-1`.
        *   Is `1.0.0-alpha` < `1.0.0-alpha-1`?
        *   Wait, `alpha-1` is one identifier. `alpha` is one identifier.
        *   If I have `1.0.0-alpha` and `1.0.0-alpha-1`.
        *   `ids_a` = `['alpha']`. `ids_b` = `['alpha-1']`.
        *   Loop i=0. `id_a`='alpha', `id_b`='alpha-1'.
        *   Both alphanumeric.
        *   'alpha' < 'alpha-1' (since 'alpha' is prefix).
        *   So `1.0.0-alpha` < `1.0.0-alpha-1`.
        *   Is this correct according to SemVer?
        *   SemVer: "Pre-release versions have a lower precedence than the associated normal version."
        *   But comparing two pre-releases?
        *   "Identifiers consisting of only digits are compared numerically and identifiers with letters or hyphens are compared lexically in ASCII sort order."
        *   "Numeric identifiers always have lower precedence than alphanumeric identifiers."
        *   So yes, lexical comparison applies. `alpha` < `alpha-1`.
        *   Wait, is `alpha-1` valid? Yes.
        *   Is `alpha` < `alpha-1`? Yes.
        *   However, usually people write `1.0.0-alpha.1` (dot separator).
        *   If they write `1.0.0-alpha-1`, it's treated as one chunk.
        *   My code handles this correctly.

    *   *One Logic Check:* `isdigit()` on string with hyphen?
        *   `isdigit()` returns False for "alpha-1". Correct.
        *   It returns True for "123". Correct.

10. **Final Polish:**
    The code looks solid. The tests cover the requirements.
    I will structure the output clearly with the function first, then the tests.

    One small detail: The prompt asks for "6 unit tests". I will group them into a class or just a list of assertions. A `unittest` class is standard.

    Let's double check the `isdigit` check for alphanumeric identifiers.
    SemVer spec: "Identifiers MUST comprise only ASCII alphanumerics and hyphens [0-9A-Za-z-]."
    So an identifier is either purely digits (numeric) or contains at least one letter/hyphen (alphanumeric).
    `isdigit()` checks if *all* characters are digits.
    If `isdigit()` is False, it contains a non-digit (letter or hyphen), so it is alphanumeric.
    This logic holds.

    One edge case: `1.0.0-01` vs `1.0.0-1`.
    `01` is numeric (digits). `1` is numeric.
    `int("01")` is 1. `int("1")` is 1.
    They are equal.
    SemVer says leading zeros are not allowed, but if present, they are usually treated as numbers in comparison logic unless strict validation is enforced. The prompt implies comparison logic, not validation.

    Final check on imports. `unittest` is standard library.

    The prompt asks for "Implement a function... Include 6 unit tests".
    I will provide the code block containing both.

    Let's verify the test cases requested one last time.
    1. Pre-release vs release. (Covered)
    2. Numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta). (Covered)
    3. Differing field counts (1.0.0-alpha vs 1.0.0-alpha.1). (Covered)
    4. Build metadata ignored. (Covered)
    5. (Extra) Basic version comparison.
    6. (Extra) Mixed comparison.

    Actually, looking at the list, maybe I should make sure the 6 tests are distinct and cover the requirements specifically.
    1. `compare_versions("1.0.0-alpha", "1.0.0")` -> -1 (Pre-release vs Release)
    2. `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1 (Numeric vs Alphanumeric ID)
    3. `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1 (Differing field counts)
    4. `compare_versions("1.0.0+build.1", "1.0.0+build.2")` -> 0 (Build metadata ignored)
    5. `compare_versions("1.0.0", "1.0.1")` -> -1 (Patch comparison)
    6. `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.2")` -> -1 (Numeric ID comparison)

    That looks like a solid set of 6.

    Wait, test 2 in my list above `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
    `1` is numeric. `beta` is alphanumeric.
    Rule: Numeric < Alphanumeric.
    So `1.0.0-alpha.1` < `1.0.0-alpha.beta`.
    Result -1.
    My code: `is_num_a` (True) and `not is_num_b` (True) -> returns -1. Correct.

    Test 3: `1.0.0-alpha` vs `1.0.0-alpha.1`.
    `ids_a` = `['alpha']`, `ids_b` = `['alpha', '1']`.
    Loop i=0: `alpha` == `alpha`.
    Loop ends.
    Length check: len 1 < len 2.
    Returns -1. Correct.

    Test 4: `1.0.0+build.1` vs `1.0.0+build.2`.
    Strips `+`. Both `1.0.0`.
    Pre-release None.
    Returns 0. Correct.

    Looks good.

    One distinct detail: The prompt asks for `compare_versions(a: str, b: str) -> int`.
    I will stick to that signature.

    Let's write out the solution.

    (Self-Correction during drafting):
    In Python `str.isdigit()` might return True for superscript numbers in Unicode, but for standard ASCII inputs it's fine. To be strictly compliant with ASCII SemVer, checking `all(c in '0123456789' for c in s)` is safer, but `isdigit()` is generally accepted for this context. I'll stick to `isdigit()` for readability, or maybe a helper `is_numeric`. Actually `isdigit()` is fine.

    Wait, `isdigit()` returns True for `'²'` (superscript 2). SemVer is ASCII only.
    A safer check for SemVer numeric identifiers is `all(c.isdigit() for c in s)`? No, `isdigit` on the whole string is the standard check.
    However, to be 100% safe against weird unicode inputs if the user passes them (though SemVer is ASCII), `re.match(r'^\d+$', s)` is the most robust way to check if a string is a non-negative integer.
    But given the context, `isdigit()` is likely sufficient. I will use `isdigit()` as it's standard Pythonic.

    Actually, let's look at `isdigit()` docs. "Return true if all characters in the string are digits and there is at least one character, false otherwise."
    Digits include decimal characters and digit characters that do not have the Unicode Numeric value property.
    For ASCII, it's 0-9.
    It's fine.

    One final check on logic flow.
    Input: `1.0.0-0` vs `1.0.0-00`.
    `0` is numeric. `00` is numeric.
    `int('0')` = 0. `int('00')` = 0.
    Equal.
    SemVer: `0` and `00` are numerically equal.
    (Though `00` is invalid SemVer, comparison logic holds).

    Input: `1.0.0-abc` vs `1.0.0-ABC`.
    Alphanumeric.
    'abc' vs 'ABC'.
    ASCII: 'A' (65) < 'a' (97).
    So `1.0
