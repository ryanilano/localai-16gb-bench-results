# 35B_UD-Q3_K_M — 01_semver_compare

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

The user wants a Python function `compare_versions(a: str, b: str) -> int` that implements Semantic Versioning 2.0.0 comparison logic.

**SemVer 2.0.0 Rules:**
1.  **Version Core:** `MAJOR.MINOR.PATCH`
2.  **Pre-release:** `-` followed by dot-separated identifiers.
3.  **Build Metadata:** `+` followed by identifiers (ignored for precedence).
4.  **Comparison Logic:**
    *   Compare MAJOR, MINOR, PATCH numerically.
    *   If core versions are equal, compare Pre-release.
    *   **Pre-release Precedence:**
        *   Pre-release has lower precedence than release (no pre-release).
        *   If both have pre-release, compare identifiers dot-by-dot.
        *   Identifiers are ASCII strings.
        *   Numeric identifiers (all digits) are compared numerically.
        *   Non-numeric identifiers are compared lexically (ASCII sort order).
        *   Numeric identifiers always have lower precedence than alphanumeric identifiers.
        *   A larger set of identifiers has higher precedence if all preceding identifiers are equal.

**Implementation Plan:**
1.  **Parsing:**
    *   Strip build metadata (everything after `+`).
    *   Split into core version (e.g., `1.2.3`) and pre-release (e.g., `alpha.1`).
    *   Parse core version into integers.
    *   Split pre-release into a list of identifiers.
2.  **Comparison Function:**
    *   Compare core versions (major, minor, patch). If unequal, return result.
    *   Handle pre-release logic:
        *   Case 1: `a` has no pre-release, `b` has pre-release -> `a > b` (Return 1).
        *   Case 2: `a` has pre-release, `b` has no pre-release -> `a < b` (Return -1).
        *   Case 3: Both have pre-release or neither has pre-release.
            *   If neither has pre-release -> Return 0.
            *   Compare identifiers list element by element.
            *   Identify type of identifier (numeric vs alphanumeric).
            *   Compare based on type and value.
            *   If one list runs out before the other, the longer one is greater.

**Detailed Pre-release Comparison Logic:**
Let `id_a` and `id_b` be two identifiers.
*   Check if `id_a` is numeric (all digits).
*   Check if `id_b` is numeric (all digits).
*   If both numeric: compare `int(id_a)` vs `int(id_b)`.
*   If one numeric, one alphanumeric: Numeric is lower.
*   If both alphanumeric: compare strings `id_a` vs `id_b` (lexicographical).

**Edge Cases:**
*   `1.0.0-alpha` vs `1.0.0-alpha.1` (Length difference).
*   `1.0.0-1` vs `1.0.0-alpha` (Numeric vs Alpha).
*   `1.0.0-alpha` vs `1.0.0` (Pre vs Release).
*   `1.0.0+build` vs `1.0.0` (Metadata ignored).

**Unit Tests:**
1.  Pre-release vs Release (`1.0.0-alpha` < `1.0.0`).
2.  Numeric vs Alphanumeric (`1.0.0-1` < `1.0.0-alpha`).
3.  Differing field counts (`1.0.0-alpha` < `1.0.0-alpha.1`).
4.  Build metadata ignored (`1.0.0+build` == `1.0.0`).
5.  Standard numeric comparison (`1.0.0` < `2.0.0`).
6.  Alphanumeric comparison (`1.0.0-alpha.beta` vs `1.0.0-beta`).

Let's write the code.

```python
def compare_versions(a: str, b: str) -> int:
    # 1. Strip build metadata (everything after '+')
    def strip_metadata(version_str):
        if '+' in version_str:
            return version_str.split('+')[0]
        return version_str

    a_clean = strip_metadata(a)
    b_clean = strip_metadata(b)

    # 2. Split into core (MAJOR.MINOR.PATCH) and pre-release
    def parse_version(version_str):
        # Split by '-' to separate core and pre-release
        parts = version_str.split('-', 1)
        core_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # Parse core version
        try:
            major, minor, patch = map(int, core_str.split('.'))
        except ValueError:
            raise ValueError(f"Invalid version format: {version_str}")
            
        # Parse pre-release identifiers
        pre_release_ids = []
        if pre_release_str:
            pre_release_ids = pre_release_str.split('.')
            
        return (major, minor, patch), pre_release_ids

    core_a, pre_a = parse_version(a_clean)
    core_b, pre_b = parse_version(b_clean)

    # 3. Compare Core Version
    for ca, cb in zip(core_a, core_b):
        if ca < cb: return -1
        if ca > cb: return 1
    
    # 4. Compare Pre-release
    # Rule: No pre-release has higher precedence than pre-release
    has_pre_a = len(pre_a) > 0
    has_pre_b = len(pre_b) > 0
    
    if has_pre_a and not has_pre_b:
        return -1 # a < b
    if not has_pre_a and has_pre_b:
        return 1  # a > b
    
    # If both have no pre-release (or both have same empty list logic if handled generally), they are equal so far
    if not has_pre_a and not has_pre_b:
        return 0
    
    # Both have pre-release, compare identifiers
    # Precedence: Compare dot-separated identifiers
    len_a = len(pre_a)
    len_b = len(pre_b)
    min_len = min(len_a, len_b)
    
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b[i]
        
        # Check if numeric
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()
        
        if is_num_a and is_num_b:
            # Compare numerically
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a < val_b: return -1
            if val_a > val_b: return 1
        elif is_num_a:
            # Numeric < Alphanumeric
            return -1
        elif is_num_b:
            # Alphanumeric > Numeric
            return 1
        else:
            # Both alphanumeric, compare lexically
            if id_a < id_b: return -1
            if id_a > id_b: return 1
            
    # If all compared identifiers are equal, the longer set has higher precedence
    if len_a < len_b: return -1
    if len_a > len_b: return 1
    
    return 0
```

Let's double check the logic against specific rules.
- "Numeric identifiers always rank lower than alphanumeric ones".
  My logic: `elif is_num_a: return -1` (since `id_a` is num, `id_b` is not). This is correct.
- "A larger set of fields ranks higher when all preceding ones are equal".
  My logic: checks lengths after loop. Correct.
- "Build metadata (after '+') is IGNORED".
  My logic: `strip_metadata` handles this. Correct.

Let's refine the parsing slightly to be robust. `isdigit()` works for "1", "10", etc. Does it handle negative numbers? SemVer identifiers are non-negative. `1.0.0-1` is valid. `1.0.0-01` is valid (leading zeros allowed in pre-release? SemVer spec says "Identifiers MUST comprise only ASCII alphanumerics and hyphen [0-9A-Za-z-]. Identifiers MUST NOT be empty. Numeric identifiers MUST NOT include leading zeroes." Wait, actually the spec says "Numeric identifiers MUST NOT include leading zeroes" implies `01` is invalid per strict SemVer, but usually parsers accept it or treat it as string. However, the prompt asks to implement the rules. If I strictly follow "Numeric identifiers MUST NOT include leading zeroes", I might need to validate, but usually for comparison logic, treating `01` as numeric `1` is safer or just string compare if it fails `isdigit`. Actually `isdigit` returns True for `01`. `int('01')` is 1. If I have `01` vs `1`, `int` compares them equal. If I have `01` vs `02`, `int` compares 1 vs 2. This seems consistent.

Let's write the unit tests.
1. `1.0.0-alpha` vs `1.0.0` -> -1
2. `1.0.0-alpha.1` vs `1.0.0-alpha.beta` -> -1 (Numeric 1 < Alpha 'beta')
   Wait, the prompt asked for `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
   `alpha` == `alpha`.
   Next: `1` (numeric) vs `beta` (alphanumeric).
   Numeric < Alphanumeric. So `1.0.0-alpha.1` < `1.0.0-alpha.beta`. Result -1.
3. `1.0.0-alpha` vs `1.0.0-alpha.1` -> -1 (Length 1 < Length 2).
4. `1.0.0+build` vs `1.0.0` -> 0.
5. `1.0.0` vs `2.0.0` -> -1.
6. `1.0.0-alpha` vs `1.0.0-alpha` -> 0.

Wait, the prompt asks for specific cases.
- "pre-release vs release"
- "numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta)"
- "differing field counts (1.0.0-alpha vs 1.0.0-alpha.1)"
- "build metadata is ignored"

I will structure the tests to cover these explicitly.

One detail: `isdigit()` vs `isnumeric()`. `isdigit()` is sufficient for ASCII digits.
One detail: Lexical comparison. `alpha` vs `beta`. `a` < `b`. Correct.

Let's refine the code structure to be clean and include the tests.

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    # 1. Strip build metadata (everything after '+')
    def strip_metadata(version_str):
        if '+' in version_str:
            return version_str.split('+')[0]
        return version_str

    a_clean = strip_metadata(a)
    b_clean = strip_metadata(b)

    # 2. Split into core (MAJOR.MINOR.PATCH) and pre-release
    def parse_version(version_str):
        # Split by '-' to separate core and pre-release
        # Use maxsplit=1 to handle cases where pre-release might contain hyphens? 
        # Actually SemVer identifiers can contain hyphens. 
        # The version string format is MAJOR.MINOR.PATCH[-PRERELEASE].
        # The hyphen separating core and pre-release is the first one.
        parts = version_str.split('-', 1)
        core_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # Parse core version
        try:
            major, minor, patch = map(int, core_str.split('.'))
        except ValueError:
            raise ValueError(f"Invalid version format: {version_str}")
            
        # Parse pre-release identifiers
        pre_release_ids = []
        if pre_release_str:
            # Identifiers are separated by dots
            pre_release_ids = pre_release_str.split('.')
            
        return (major, minor, patch), pre_release_ids

    core_a, pre_a = parse_version(a_clean)
    core_b, pre_b = parse_version(b_clean)

    # 3. Compare Core Version (Major, Minor, Patch)
    for ca, cb in zip(core_a, core_b):
        if ca < cb: return -1
        if ca > cb: return 1
    
    # 4. Compare Pre-release
    # Rule: No pre-release has higher precedence than pre-release
    has_pre_a = len(pre_a) > 0
    has_pre_b = len(pre_b) > 0
    
    if has_pre_a and not has_pre_b:
        return -1 # a < b
    if not has_pre_a and has_pre_b:
        return 1  # a > b
    
    # If both have no pre-release (or both have same empty list logic if handled generally), they are equal so far
    if not has_pre_a and not has_pre_b:
        return 0
    
    # Both have pre-release, compare identifiers
    # Precedence: Compare dot-separated identifiers
    len_a = len(pre_a)
    len_b = len(pre_b)
    min_len = min(len_a, len_b)
    
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b[i]
        
        # Check if numeric (all digits)
        # SemVer spec says identifiers are ASCII alphanumerics and hyphen.
        # Numeric identifiers are those consisting of digits only.
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()
        
        if is_num_a and is_num_b:
            # Compare numerically
            # Note: SemVer says numeric identifiers MUST NOT include leading zeroes.
            # We assume valid input or treat as int.
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a < val_b: return -1
            if val_a > val_b: return 1
        elif is_num_a:
            # Numeric < Alphanumeric
            return -1
        elif is_num_b:
            # Alphanumeric > Numeric
            return 1
        else:
            # Both alphanumeric, compare lexically (ASCII order)
            if id_a < id_b: return -1
            if id_a > id_b: return 1
            
    # If all compared identifiers are equal, the longer set has higher precedence
    if len_a < len_b: return -1
    if len_a > len_b: return 1
    
    return 0

# Unit Tests
class TestSemVerComparison(unittest.TestCase):
    
    def test_pre_release_vs_release(self):
        # 1.0.0-alpha should be less than 1.0.0
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        # 1.0.0-alpha.1 vs 1.0.0-alpha.beta
        # 'alpha' == 'alpha'
        # '1' (numeric) < 'beta' (alphanumeric)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1"), 1)

    def test_differing_field_counts(self):
        # 1.0.0-alpha vs 1.0.0-alpha.1
        # 'alpha' == 'alpha'
        # a has no next field, b has '1'
        # Larger set ranks higher
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

    def test_build_metadata_ignored(self):
        # 1.0.0+build vs 1.0.0
        self.assertEqual(compare_versions("1.0.0+build", "1.0.0"), 0)
        self.assertEqual(compare_versions("1.0.0", "1.0.0+build"), 0)
        # Also check if metadata affects order if core/pre are equal
        self.assertEqual(compare_versions("1.0.0-alpha+build1", "1.0.0-alpha+build2"), 0)

    def test_standard_numeric_comparison(self):
        # 1.0.0 vs 2.0.0
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.1.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.1"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0"), 0)

    def test_complex_pre_release(self):
        # 1.0.0-alpha.1 vs 1.0.0-alpha.beta
        # Wait, I did this in test 2. Let's do a different one.
        # 1.0.0-1 vs 1.0.0-alpha
        # Numeric 1 < Alpha 'alpha'
        self.assertEqual(compare_versions("1.0.0-1", "1.0.0-alpha"), -1)

if __name__ == '__main__':
    unittest.main()
```

Wait, `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
`alpha` vs `alpha` -> equal.
`1` vs `beta`. `1` is numeric. `beta` is alphanumeric. Numeric < Alphanumeric. So `1` < `beta`. Result -1.
My code logic:
`is_num_a` (for '1') is True.
`is_num_b` (for 'beta') is False.
`elif is_num_a: return -1`. Correct.

Wait, `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
Is `beta` alphanumeric? Yes.
Is `1` numeric? Yes.
So `1` < `beta`.
Correct.

Let's check `1.0.0-alpha` vs `1.0.0-alpha.1`.
`alpha` vs `alpha` -> equal.
`pre_a` ends. `pre_b` has `1`.
Loop finishes.
`len_a` (1) < `len_b` (2).
Return -1. Correct.

Let's check `1.0.0+build` vs `1.0.0`.
`strip_metadata` removes `+build`.
Both become `1.0.0`.
Core equal.
Pre equal (empty).
Return 0. Correct.

Let's check `1.0.0-1` vs `1.0.0-alpha`.
`1` vs `alpha`.
`1` is numeric. `alpha` is alphanumeric.
Return -1. Correct.

One edge case: `1.0.0` vs `1.0.0`.
Core equal. Pre equal (empty). Return 0. Correct.

One edge case: `1.0.0-alpha` vs `1.0.0-alpha`.
Core equal. Pre equal. Loop runs once. `alpha` vs `alpha`. Equal.
Lengths equal. Return 0. Correct.

One edge case: `1.0.0-alpha.1` vs `1.0.0-alpha.1`.
Core equal. Pre equal. Loop runs twice. Equal. Lengths equal. Return 0. Correct.

One edge case: `1.0.0-alpha.beta` vs `1.0.0-alpha.beta`.
Equal.

One edge case: `1.0.0-alpha.1` vs `1.0.0-alpha.01`.
`1` vs `01`.
Both numeric. `int('1')` vs `int('01')`. 1 vs 1. Equal.
Loop continues.
Lengths equal. Return 0.
Wait, SemVer says "Numeric identifiers MUST NOT include leading zeroes." So `01` is technically invalid input for strict SemVer. But if provided, treating it as numeric 1 is the standard interpretation for version comparison unless validation is required. The prompt doesn't ask for validation, just comparison logic.

One detail: `isdigit()` check.
`"123".isdigit()` -> True.
`"12a".isdigit()` -> False.
`"12-".isdigit()` -> False.
This seems correct for identifying numeric identifiers.

Let's verify the imports. `unittest` is standard.
The function signature is `compare_versions(a: str, b: str) -> int`.

I should make sure the code is contained in the solution.
I will write the solution clearly.

One thought: The prompt asks for "6 unit tests". I have:
1. Pre-release vs Release.
2. Numeric vs Alphanumeric.
3. Differing field counts.
4. Build metadata ignored.
5. Standard numeric comparison (Major/Minor/Patch).
6. Complex pre-release (e.g. 1.0.0-1 vs 1.0.0-alpha).

This covers the requirements.

Final check on logic:
- `a < b` returns -1.
- `a == b` returns 0.
- `a > b` returns 1.

Code structure:
```python
import unittest

def compare_versions(a: str, b: str) -> int:
    # ... implementation ...
    pass

# ... tests ...
```

I'll put the implementation inside the solution block.

One specific detail about SemVer comparison of identifiers:
"If two identifiers are numeric, they are compared as integers."
"If one is numeric and the other is not, the numeric one has lower precedence."
"If both are not numeric, they are compared lexically."

My code:
```python
        if is_num_a and is_num_b:
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a < val_b: return -1
            if val_a > val_b: return 1
        elif is_num_a:
            return -1
        elif is_num_b:
            return 1
        else:
            if id_a < id_b: return -1
            if id_a > id_b: return 1
```
This matches perfectly.

One minor point: `split('-', 1)` handles `1.0.0-alpha-beta`.
`core_str` = `1.0.0`.
`pre_release_str` = `alpha-beta`.
`pre_release_ids` = `['alpha-beta']`.
`isdigit()` on `alpha-beta` is False.
So it treats `alpha-beta` as alphanumeric.
This is correct.

What about `1.0.0-alpha.1-beta`?
`pre_release_ids` = `['alpha', '1', 'beta']`.
Comparison works dot by dot. Correct.

Double check the `strip_metadata` logic.
`1.0.0+build` -> `1.0.0`.
`1.0.0-alpha+build` -> `1.0.0-alpha`.
Correct.

The prompt requires "Include 6 unit tests". I will include them in the code block.

One small thing: `isdigit()` might behave differently in Python 2 vs 3 regarding unicode, but SemVer is ASCII. `isdigit()` is fine for ASCII digits. `isdecimal()` is stricter but `isdigit()` is generally accepted for version numbers.

Let's finalize the code.

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    # 1. Strip build metadata (everything after '+')
    def strip_metadata(version_str):
        if '+' in version_str:
            return version_str.split('+')[0]
        return version_str

    a_clean = strip_metadata(a)
    b_clean = strip_metadata(b)

    # 2. Split into core (MAJOR.MINOR.PATCH) and pre-release
    def parse_version(version_str):
        # Split by '-' to separate core and pre-release
        # maxsplit=1 ensures we only split on the first hyphen
        parts = version_str.split('-', 1)
        core_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # Parse core version
        try:
            major, minor, patch = map(int, core_str.split('.'))
        except ValueError:
            raise ValueError(f"Invalid version format: {version_str}")
            
        # Parse pre-release identifiers
        pre_release_ids = []
        if pre_release_str:
            # Identifiers are separated by dots
            pre_release_ids = pre_release_str.split('.')
            
        return (major, minor, patch), pre_release_ids

    core_a, pre_a = parse_version(a_clean)
    core_b, pre_b = parse_version(b_clean)

    # 3. Compare Core Version (Major, Minor, Patch)
    for ca, cb in zip(core_a, core_b):
        if ca < cb: return -1
        if ca > cb: return 1
    
    # 4. Compare Pre-release
    # Rule: No pre-release has higher precedence than pre-release
    has_pre_a = len(pre_a) > 0
    has_pre_b = len(pre_b) > 0
    
    if has_pre_a and not has_pre_b:
        return -1 # a < b
    if not has_pre_a and has_pre_b:
        return 1  # a > b
    
    # If both have no pre-release, they are equal
    if not has_pre_a and not has_pre_b:
        return 0
    
    # Both have pre-release, compare identifiers
    len_a = len(pre_a)
    len_b = len(pre_b)
    min_len = min(len_a, len_b)
    
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b[i]
        
        # Check if numeric (all digits)
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()
        
        if is_num_a and is_num_b:
            # Compare numerically
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a < val_b: return -1
            if val_a > val_b: return 1
        elif is_num_a:
            # Numeric < Alphanumeric
            return -1
        elif is_num_b:
            # Alphanumeric > Numeric
            return 1
        else:
            # Both alphanumeric, compare lexically (ASCII order)
            if id_a < id_b: return -1
            if id_a > id_b: return 1
            
    # If all compared identifiers are equal, the longer set has higher precedence
    if len_a < len_b: return -1
    if len_a > len_b: return 1
    
    return 0

# Unit Tests
class TestSemVerComparison(unittest.TestCase):
    
    def test_pre_release_vs_release(self):
        # Pre-release has lower precedence than release
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        # Numeric identifiers (1) have lower precedence than alphanumeric (beta)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1"), 1)

    def test_differing_field_counts(self):
        # A larger set of fields ranks higher when all preceding ones are equal
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

    def test_build_metadata_ignored(self):
        # Build metadata (after '+') is ignored in precedence
        self.assertEqual(compare_versions("1.0.0+build", "1.0.0"), 0)
        self.assertEqual(compare_versions("1.0.0", "1.0.0+build"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+build1", "1.0.0-alpha+build2"), 0)

    def test_standard_numeric_comparison(self):
        # Compare major, minor, patch numerically
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.1.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.1"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0"), 0)

    def test_alphanumeric_lexical_comparison(self):
        # Alphanumeric identifiers compared lexically in ASCII order
        # 'alpha' < 'beta'
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-beta"), -1)
        self.assertEqual(compare_versions("1.0.0-beta", "1.0.0-alpha"), 1)
        # '1' < '2' (numeric)
        self.assertEqual(compare_versions("1.0.0-1", "1.0.0-2"), -1)

if __name__ == '__main__':
    unittest.main()
```
The test `test_alphanumeric_lexical_comparison` adds the 6th test case (or rather, the 6th distinct test method). The prompt asked for 6 unit tests covering specific cases. I have covered all requested cases plus general numeric/alpha comparison.

Wait, `1.0.0-alpha.1` vs `1.0.0-alpha.beta` is covered in `test_numeric_vs_alphanumeric_identifiers`.
`1.0.0-alpha` vs `1.0.0-alpha.1` is covered in `test_differing_field_counts`.
`1.0.0-alpha` vs `1.0.0` is covered in `test_pre_release_vs_release`.
`1.0.0+build` vs `1.0.0` is covered in `test_build_metadata_ignored`.

I have 6 test methods.
1. `test_pre_release_vs_release`
2. `test_numeric_vs_alphanumeric_identifiers`
3. `test_differing_field_counts`
4. `test_build_metadata_ignored`
5. `test_standard_numeric_comparison`
6. `test_alphanumeric_lexical_comparison`

This looks solid.

One small detail: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
`alpha` == `alpha`.
`1` vs `beta`.
`1` is numeric. `beta` is alphanumeric.
Numeric < Alphanumeric.
So `1.0.0-alpha.1` < `1.0.0-alpha.beta`.
Result -1.
My code returns -1.
Correct.

The user prompt says "e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta".
This implies checking that `1` comes before `beta`.
My test checks this.

One potential issue: `isdigit()` returns `False` for empty strings.
If `pre_release_str` is `alpha..beta` (double dot), `split('.')` might produce empty strings.
SemVer spec: "Identifiers MUST NOT be empty."
If input is invalid, behavior is undefined, but my code would crash or behave weirdly.
`id_a.isdigit()` on empty string is False.
`id_a` is `""`.
`id_b` is `"beta"`.
`is_num_a` False, `is_num_b` False.
`""` < `"beta"` -> True.
It would return -1.
This is acceptable for "garbage in" unless strict validation is requested. The prompt asks to implement precedence rules, implying valid inputs or at least standard parsing.

I will proceed with this implementation.

Final check on imports: `unittest` is standard library. No external dependencies.
The function signature matches.
The return values match.
The precedence rules are followed.

One tiny optimization: `zip(core_a, core_b)` works because `core_a` and `core_b` are tuples of length 3.
`zip` stops at shortest, which is fine here.

Double check: `1.0.0` vs `1.0.0.0`?
SemVer is strictly `MAJOR.MINOR.PATCH`. `1.0.0.0` is invalid.
My parser `map(int, core_str.split('.'))` would fail if there are 4 parts.
`ValueError` raised. This is acceptable behavior for invalid input.

The solution seems robust enough for the requirements.

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    # 1. Strip build metadata (everything after '+')
    def strip_metadata(version_str):
        if '+' in version_str:
            return version_str.split('+')[0]
        return version_str

    a_clean = strip_metadata(a)
    b_clean = strip_metadata(b)

    # 2. Split into core (MAJOR.MINOR.PATCH) and pre-release
    def parse_version(version_str):
        # Split by '-' to separate core and pre-release
        # maxsplit=1 ensures we only split on the first hyphen
        parts = version_str.split('-', 1)
        core_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # Parse core version
        try:
            major, minor, patch = map(int, core_str.split('.'))
        except ValueError:
            raise ValueError(f"Invalid version format: {version_str}")
            
        # Parse pre-release identifiers
        pre_release_ids = []
        if pre_release_str:
            # Identifiers are separated by dots
            pre_release_ids = pre_release_str.split('.')
            
        return (major, minor, patch), pre_release_ids

    core_a, pre_a = parse_version(a_clean)
    core_b, pre_b = parse_version(b_clean)

    # 3. Compare Core Version (Major, Minor, Patch)
    for ca, cb in zip(core_a, core_b):
        if ca < cb: return -1
        if ca > cb: return 1
    
    # 4. Compare Pre-release
    # Rule: No pre-release has higher precedence than pre-release
    has_pre_a = len(pre_a) > 0
    has_pre_b = len(pre_b) > 0
    
    if has_pre_a and not has_pre_b:
        return -1 # a < b
    if not has_pre_a and has_pre_b:
        return 1  # a > b
    
    # If both have no pre-release, they are equal
    if not has_pre_a and not has_pre_b:
        return 0
    
    # Both have pre-release, compare identifiers
    len_a = len(pre_a)
    len_b = len(pre_b)
    min_len = min(len_a, len_b)
    
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b
