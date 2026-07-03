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

The user wants a Python function `compare_versions` that compares two Semantic Versioning 2.0.0 strings.
The function should return -1 if `a < b`, 0 if `a == b`, and 1 if `a > b`.

SemVer 2.0.0 rules:
1.  **Format**: `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`
2.  **Major, Minor, Patch**: Compare numerically.
3.  **Pre-release**:
    *   If both have no pre-release, they are equal (ignoring build metadata).
    *   If one has pre-release and the other doesn't, the one *without* is higher (release > pre-release).
    *   If both have pre-release, compare dot-separated identifiers.
4.  **Pre-release Identifiers**:
    *   Split by `.`.
    *   Compare identifier by identifier.
    *   If both are numeric, compare as integers.
    *   If both are alphanumeric (string), compare as strings (ASCII order).
    *   If one is numeric and the other is alphanumeric, numeric is lower (smaller) than alphanumeric.
    *   If identifiers are equal, move to the next one.
    *   If all compared identifiers are equal, the version with more identifiers is higher (longer set > shorter set).
5.  **Build Metadata**: Ignored completely for precedence.

**Steps to implement:**

1.  **Parsing**:
    *   Split off build metadata first (split on `+`). We only care about the part before `+`.
    *   Split off pre-release (split on `-`). We only care about the part before `-` for the core version (major.minor.patch), and the part after `-` for pre-release logic.
    *   Parse core version (major.minor.patch) into integers.

2.  **Comparison Logic**:
    *   Compare core versions (major, minor, patch). If not equal, return result.
    *   Determine pre-release status.
        *   Case 1: Both have no pre-release. Return 0.
        *   Case 2: `a` has pre-release, `b` does not. `a` is lower. Return -1.
        *   Case 3: `a` does not have pre-release, `b` has pre-release. `a` is higher. Return 1.
        *   Case 4: Both have pre-release. Compare identifiers.

3.  **Pre-release Comparison Logic**:
    *   Split pre-release strings by `.`.
    *   Iterate through identifiers.
    *   Check type (numeric vs string).
    *   Apply comparison rules.
    *   Handle length differences (shorter < longer if prefixes match).

4.  **Unit Tests**:
    *   Need 6 tests covering specific scenarios.
    *   Test 1: Basic equality (1.0.0 == 1.0.0).
    *   Test 2: Numeric comparison (1.0.0 < 2.0.0).
    *   Test 3: Pre-release vs Release (1.0.0-alpha < 1.0.0).
    *   Test 4: Numeric vs Alphanumeric identifiers (1.0.0-alpha.1 < 1.0.0-alpha.beta).
    *   Test 5: Differing field counts (1.0.0-alpha < 1.0.0-alpha.1).
    *   Test 6: Build metadata ignored (1.0.0+build1 == 1.0.0+build2).

Let's refine the parsing and comparison logic.

**Parsing**:
Input: `a_str`, `b_str`.

Function `parse_version(version_str)`:
1.  Remove build metadata: `core_pre = version_str.split('+')[0]`.
2.  Split pre-release: `parts = core_pre.split('-', 1)`.
    *   `core_version_str = parts[0]`
    *   `pre_release_str = parts[1]` if len(parts) > 1 else None
3.  Parse core: `core_nums = [int(x) for x in core_version_str.split('.')]`.
    *   Return `(core_nums, pre_release_str)`.

**Comparison**:
1.  Parse `a` and `b`.
2.  Compare `core_nums` element by element.
    *   `a_nums = [major, minor, patch]`
    *   `b_nums = [major, minor, patch]`
    *   Loop `i` from 0 to 2:
        *   if `a_nums[i] < b_nums[i]` return -1
        *   if `a_nums[i] > b_nums[i]` return 1
3.  Handle pre-release.
    *   `a_pre = a_parsed[1]`
    *   `b_pre = b_parsed[1]`
    *   If `a_pre` is None and `b_pre` is None: return 0.
    *   If `a_pre` is not None and `b_pre` is None: return -1 (Pre-release < Release).
    *   If `a_pre` is None and `b_pre` is not None: return 1 (Release > Pre-release).
    *   If both not None:
        *   `a_idents = a_pre.split('.')`
        *   `b_idents = b_pre.split('.')`
        *   Loop `i` from 0 to `min(len(a_idents), len(b_idents)) - 1`:
            *   `ident_a = a_idents[i]`
            *   `ident_b = b_idents[i]`
            *   Determine types.
                *   `is_num_a = ident_a.isdigit()` (Note: SemVer says numeric identifiers MUST NOT include leading zeroes, but `isdigit()` is a safe check for numeric logic usually, though strictly speaking we should check if it's a valid number. The prompt implies standard SemVer strings).
                *   `is_num_b = ident_b.isdigit()`
            *   Comparison logic:
                *   If `is_num_a` and `is_num_b`:
                    *   Compare `int(ident_a)` vs `int(ident_b)`.
                *   Else if `is_num_a` and not `is_num_b`:
                    *   Numeric is lower. Return -1.
                *   Else if not `is_num_a` and `is_num_b`:
                    *   Alphanumeric is higher. Return 1.
                *   Else (both alphanumeric):
                    *   Compare strings `ident_a` vs `ident_b`.
                    *   If `ident_a < ident_b` return -1.
                    *   If `ident_a > ident_b` return 1.
            *   If loop finishes without returning, identifiers are equal so far.
        *   Check lengths.
            *   If `len(a_idents) < len(b_idents)`: return -1 (Shorter set < Longer set).
            *   If `len(a_idents) > len(b_idents)`: return 1.
        *   Return 0.

**Corner Cases / Details**:
*   `isdigit()` behavior: "10".isdigit() is True. "10a".isdigit() is False. This works.
*   Leading zeros: SemVer says numeric identifiers must not have leading zeros (except 0). The prompt doesn't explicitly ask to validate this, just compare. Assuming valid inputs or treating them as strings if they have leading zeros? Actually, `isdigit` returns true for "01". `int("01")` is 1. SemVer spec says "Numeric identifiers MUST NOT include leading zeroes". If input violates this, behavior is undefined by spec, but usually treated as string or parsed as int. I will assume valid SemVer inputs.
*   Empty pre-release? e.g. "1.0.0-". Usually invalid, but split might result in empty string. Empty string is not numeric, it's alphanumeric (empty string). Comparison logic should hold.

**Unit Tests Selection**:
1.  `compare_versions("1.0.0", "2.0.0")` -> -1
2.  `compare_versions("1.0.0-alpha", "1.0.0")` -> -1 (Pre-release vs Release)
3.  `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1 (Differing field counts: shorter < longer)
4.  `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1 (Numeric vs Alphanumeric: numeric < alphanumeric)
5.  `compare_versions("1.0.0+build1", "1.0.0+build2")` -> 0 (Build metadata ignored)
6.  `compare_versions("1.0.0-alpha.beta", "1.0.0-beta")` -> -1 (This covers the alphanumeric comparison logic properly: 'alpha' < 'beta', but wait. 'alpha' vs 'beta'. 'alpha' comes before 'beta'. So 1 < 0? No, 'alpha' < 'beta' implies -1. Let's check the rule: "alphanumeric compared lexically in ASCII order". 'a' (97) < 'b' (98). So alpha < beta. Result -1.
    *   Wait, the prompt asks for specific tests.
    *   "Numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta)" -> This is test 4 in my list.
    *   "Differing field counts (1.0.0-alpha vs 1.0.0-alpha.1)" -> This is test 3.
    *   "Pre-release vs release" -> Test 2.
    *   "Build metadata is ignored" -> Test 5.
    *   I need 6 tests.
    *   Test 6: Basic version comparison (1.0.0 vs 1.0.1).

Let's refine the specific tests requested in the prompt to ensure I hit the "MUST" requirements.
- Pre-release vs release: `1.0.0-alpha` vs `1.0.0`
- Numeric vs alphanumeric: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`
- Differing field counts: `1.0.0-alpha` vs `1.0.0-alpha.1`
- Build metadata ignored: `1.0.0+build1` vs `1.0.0+build2`

I need one more specific test or just general coverage to make 6.
Maybe `1.0.0-alpha.beta` vs `1.0.0-beta` (Alphanumeric comparison).
Or `1.0.0` vs `1.0.0` (Equality).

Let's write the code.

```python
def compare_versions(a: str, b: str) -> int:
    def parse_version(version_str):
        # Remove build metadata
        core_pre = version_str.split('+')[0]
        # Split pre-release
        parts = core_pre.split('-', 1)
        core_version_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # Parse core version
        try:
            core_nums = [int(x) for x in core_version_str.split('.')]
        except ValueError:
            # Fallback or error handling if needed, but assuming valid SemVer
            core_nums = [0, 0, 0] 
            
        return core_nums, pre_release_str

    a_nums, a_pre = parse_version(a)
    b_nums, b_pre = parse_version(b)

    # 1. Compare core version (major, minor, patch)
    for i in range(3):
        if a_nums[i] < b_nums[i]:
            return -1
        if a_nums[i] > b_nums[i]:
            return 1

    # 2. Compare pre-release
    # If both are None (no pre-release), they are equal
    if a_pre is None and b_pre is None:
        return 0
    
    # If one has pre-release and the other doesn't
    # Pre-release has LOWER precedence than release
    if a_pre is not None and b_pre is None:
        return -1
    if a_pre is None and b_pre is not None:
        return 1

    # Both have pre-release, compare identifiers
    a_idents = a_pre.split('.')
    b_idents = b_pre.split('.')

    # Compare identifier by identifier
    min_len = min(len(a_idents), len(b_idents))
    for i in range(min_len):
        ident_a = a_idents[i]
        ident_b = b_idents[i]
        
        # Check if numeric
        is_num_a = ident_a.isdigit()
        is_num_b = ident_b.isdigit()
        
        if is_num_a and is_num_b:
            # Both numeric, compare as integers
            val_a = int(ident_a)
            val_b = int(ident_b)
            if val_a < val_b:
                return -1
            if val_a > val_b:
                return 1
        elif is_num_a and not is_num_b:
            # Numeric is lower than alphanumeric
            return -1
        elif not is_num_a and is_num_b:
            # Alphanumeric is higher than numeric
            return 1
        else:
            # Both alphanumeric, compare lexically
            if ident_a < ident_b:
                return -1
            if ident_a > ident_b:
                return 1
    
    # If all compared identifiers are equal, the one with more fields is higher
    if len(a_idents) < len(b_idents):
        return -1
    if len(a_idents) > len(b_idents):
        return 1

    return 0
```

Let's double check the `isdigit` logic.
`"10".isdigit()` -> True.
`"10a".isdigit()` -> False.
`"alpha".isdigit()` -> False.
This works for the logic described.

Wait, check the "Numeric identifiers always rank lower than alphanumeric ones" rule.
Example: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
`a_idents` = ['alpha', '1']
`b_idents` = ['alpha', 'beta']
i=0: 'alpha' vs 'alpha'. Equal.
i=1: '1' vs 'beta'.
'1' is num. 'beta' is alpha.
Rule: Numeric < Alphanumeric.
So '1' < 'beta'.
Result: -1. Correct.

Example: `1.0.0-alpha` vs `1.0.0-alpha.1`.
`a_idents` = ['alpha']
`b_idents` = ['alpha', '1']
i=0: 'alpha' vs 'alpha'. Equal.
Loop ends.
Length check: len(a) < len(b) (1 < 2).
Result: -1. Correct.

Example: `1.0.0-alpha` vs `1.0.0`.
`a_pre` = 'alpha', `b_pre` = None.
`a_pre` is not None, `b_pre` is None.
Return -1. Correct.

Example: `1.0.0+build1` vs `1.0.0+build2`.
`a_nums` = [1,0,0], `b_nums` = [1,0,0]. Equal.
`a_pre` = None, `b_pre` = None.
Return 0. Correct.

One edge case: `1.0.0-01` vs `1.0.0-1`.
SemVer spec says numeric identifiers MUST NOT include leading zeroes.
If input is `1.0.0-01`, `isdigit` is True. `int` is 1.
If input is `1.0.0-1`, `isdigit` is True. `int` is 1.
They would be equal.
If the input is invalid SemVer (leading zeros), behavior isn't strictly defined by the prompt, but treating as integer is standard for "numeric" comparison unless specified otherwise. The prompt asks to follow SemVer precedence rules. The rule implies valid inputs. I will assume valid inputs.

Another edge case: `1.0.0-alpha.beta` vs `1.0.0-beta`.
`a_idents` = ['alpha', 'beta']
`b_idents` = ['beta']
i=0: 'alpha' vs 'beta'.
Both alpha. 'alpha' < 'beta'.
Return -1. Correct.

Looks solid.

Let's organize the unit tests.
1.  `compare_versions("1.0.0", "2.0.0")` -> -1 (Basic numeric)
2.  `compare_versions("1.0.0-alpha", "1.0.0")` -> -1 (Pre-release vs Release)
3.  `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1 (Numeric vs Alphanumeric)
4.  `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1 (Differing field counts)
5.  `compare_versions("1.0.0+build1", "1.0.0+build2")` -> 0 (Build metadata ignored)
6.  `compare_versions("1.0.0-alpha.beta", "1.0.0-beta")` -> -1 (Alphanumeric comparison)

Wait, I should check the return values for the tests.
1. -1
2. -1
3. -1
4. -1
5. 0
6. -1

Maybe include a test where `a > b`.
`compare_versions("2.0.0", "1.0.0")` -> 1.
`compare_versions("1.0.0", "1.0.0-alpha")` -> 1.
`compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1")` -> 1.

I'll include a mix to be safe, or just stick to the requested specific ones and add a generic one.
Requested:
- Pre-release vs release
- Numeric vs alphanumeric
- Differing field counts
- Build metadata ignored

I will write the solution now.

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse_version(version_str):
        # 1. Ignore Build Metadata (split on '+')
        # We only care about the part before the first '+'
        core_pre = version_str.split('+')[0]
        
        # 2. Split Pre-release (split on '-')
        # We need to distinguish between the core version and the pre-release tag
        parts = core_pre.split('-', 1)
        core_version_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # 3. Parse Major, Minor, Patch
        try:
            core_nums = [int(x) for x in core_version_str.split('.')]
        except ValueError:
            # In case of malformed version strings, though SemVer implies valid integers
            core_nums = [0, 0, 0]
            
        return core_nums, pre_release_str

    a_nums, a_pre = parse_version(a)
    b_nums, b_pre = parse_version(b)

    # 1. Compare Major, Minor, Patch numerically
    # We assume standard 3-part versions (Major.Minor.Patch)
    for i in range(3):
        if a_nums[i] < b_nums[i]:
            return -1
        if a_nums[i] > b_nums[i]:
            return 1

    # 2. Compare Pre-release
    # Rule: A version WITH a pre-release tag has LOWER precedence than the same version without one.
    
    # Case: Both have no pre-release (equal core version)
    if a_pre is None and b_pre is None:
        return 0
    
    # Case: 'a' has pre-release, 'b' does not -> 'a' is lower
    if a_pre is not None and b_pre is None:
        return -1
        
    # Case: 'a' does not have pre-release, 'b' has one -> 'a' is higher
    if a_pre is None and b_pre is not None:
        return 1

    # Case: Both have pre-release. Compare dot-by-dot.
    a_idents = a_pre.split('.')
    b_idents = b_pre.split('.')

    # Compare identifiers
    min_len = min(len(a_idents), len(b_idents))
    for i in range(min_len):
        ident_a = a_idents[i]
        ident_b = b_idents[i]
        
        # Determine if identifiers are numeric
        # Note: SemVer says numeric identifiers MUST NOT include leading zeroes.
        # We assume valid inputs or treat strings with leading zeros as numeric if they consist of digits.
        is_num_a = ident_a.isdigit()
        is_num_b = ident_b.isdigit()
        
        if is_num_a and is_num_b:
            # Both numeric: compare numerically
            val_a = int(ident_a)
            val_b = int(ident_b)
            if val_a < val_b:
                return -1
            if val_a > val_b:
                return 1
        elif is_num_a and not is_num_b:
            # Numeric identifiers always rank lower than alphanumeric ones
            return -1
        elif not is_num_a and is_num_b:
            # Alphanumeric identifiers rank higher than numeric ones
            return 1
        else:
            # Both alphanumeric: compare lexically in ASCII order
            if ident_a < ident_b:
                return -1
            if ident_a > ident_b:
                return 1
    
    # If all preceding identifiers are equal, the one with more fields ranks higher
    if len(a_idents) < len(b_idents):
        return -1
    if len(a_idents) > len(b_idents):
        return 1

    return 0
```

I will add the unit tests as comments or a separate block? The prompt asks to "Include 6 unit tests". Usually this implies a test block or print statements. I'll provide them as a test function or just print statements to demonstrate. I'll put them in a `if __name__ == "__main__":` block or just below the function.

Let's verify the tests again.
1. `compare_versions("1.0.0", "2.0.0")` -> -1.
2. `compare_versions("1.0.0-alpha", "1.0.0")` -> -1. (Pre-release < Release)
3. `compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta")` -> -1. (Numeric < Alphanumeric)
4. `compare_versions("1.0.0-alpha", "1.0.0-alpha.1")` -> -1. (Shorter < Longer)
5. `compare_versions("1.0.0+build1", "1.0.0+build2")` -> 0. (Metadata ignored)
6. `compare_versions("1.0.0-alpha.beta", "1.0.0-beta")` -> -1. (Alphanumeric comparison: 'alpha' < 'beta')

Wait, for test 6:
`a` = "1.0.0-alpha.beta" -> pre = "alpha.beta" -> idents = ["alpha", "beta"]
`b` = "1.0.0-beta" -> pre = "beta" -> idents = ["beta"]
Compare index 0: "alpha" vs "beta".
Both alphanumeric.
"alpha" < "beta" (lexically).
So `a` < `b`. Return -1.
Is this correct?
SemVer spec: "Identifiers consisting of only digits are compared numerically. Identifiers with letters or hyphens are compared lexically in ASCII sort order. Numeric identifiers always have lower precedence than non-numeric identifiers."
"alpha" vs "beta". 'a' (97) vs 'b' (98). alpha < beta.
So yes, -1 is correct.

What about `1.0.0-beta` vs `1.0.0-alpha`?
"beta" > "alpha". Return 1.

The prompt asks for "Numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta)".
My test 3 covers this exactly.

"Differing field counts (1.0.0-alpha vs 1.0.0-alpha.1)".
My test 4 covers this exactly.

"Pre-release vs release".
My test 2 covers this exactly.

"Build metadata is ignored".
My test 5 covers this exactly.

I need 6 tests.
I'll add a test for basic inequality (1.0.0 < 2.0.0) or equality (1.0.0 == 1.0.0).
Let's add `compare_versions("1.0.0", "1.0.0")` -> 0.

I will structure the code clearly.

One detail: `isdigit()` check.
If `ident_a` is "0", `isdigit` is True.
If `ident_a` is "00", `isdigit` is True.
SemVer says "Numeric identifiers MUST NOT include leading zeroes".
If the input is valid, "00" won't happen. If it does, treating it as number 0 is reasonable, or string "00".
Given the prompt asks to implement SemVer rules, and SemVer forbids leading zeros, I assume inputs follow the format. `isdigit` is the standard way to check for numeric in this context.

Final check on logic:
- `a` < `b` -> -1
- `a` == `b` -> 0
- `a` > `b` -> 1

The code looks good.

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    # Implementation...
    pass

# Unit tests
class TestSemVer(unittest.TestCase):
    def test_basic_numeric(self):
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        
    def test_pre_vs_release(self):
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        
    def test_numeric_vs_alphanumeric(self):
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        
    def test_differing_field_counts(self):
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        
    def test_build_metadata_ignored(self):
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        
    def test_alphanumeric_comparison(self):
        self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-beta"), -1)

if __name__ == "__main__":
    unittest.main()
```
The prompt asks to "Include 6 unit tests". It doesn't strictly require `unittest` framework, but it's the standard way. I'll provide the function and a script running tests.

One small check: `1.0.0-alpha` vs `1.0.0-alpha.1`.
`a_pre` = "alpha", `b_pre` = "alpha.1".
`a_idents` = ["alpha"], `b_idents` = ["alpha", "1"].
Loop runs for i=0. "alpha" == "alpha".
Loop ends.
len(a) = 1, len(b) = 2.
len(a) < len(b). Return -1.
Correct.

Double check: "1.0.0-alpha.1" vs "1.0.0-alpha.beta".
`a_idents` = ["alpha", "1"]
`b_idents` = ["alpha", "beta"]
i=0: "alpha" == "alpha".
i=1: "1" (num) vs "beta" (alpha).
Numeric < Alphanumeric. Return -1.
Correct.

The implementation seems robust.

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse_version(version_str):
        # 1. Ignore Build Metadata (everything after '+')
        # We split on '+' and take the first part.
        core_pre = version_str.split('+')[0]
        
        # 2. Split Pre-release (everything after '-')
        # We split on '-' to separate core version from pre-release tag.
        parts = core_pre.split('-', 1)
        core_version_str = parts[0]
        pre_release_str = parts[1] if len(parts) > 1 else None
        
        # 3. Parse Major, Minor, Patch numerically
        try:
            core_nums = [int(x) for x in core_version_str.split('.')]
        except ValueError:
            # Fallback for unexpected formats, though SemVer implies valid integers
            core_nums = [0, 0, 0]
            
        return core_nums, pre_release_str

    a_nums, a_pre = parse_version(a)
    b_nums, b_pre = parse_version(b)

    # 1. Compare Major, Minor, Patch numerically
    # We assume standard 3-part versions (Major.Minor.Patch)
    for i in range(3):
        if a_nums[i] < b_nums[i]:
            return -1
        if a_nums[i] > b_nums[i]:
            return 1

    # 2. Compare Pre-release
    # Rule: A version WITH a pre-release tag has LOWER precedence than the same version without one.
    
    # Case: Both have no pre-release (equal core version)
    if a_pre is None and b_pre is None:
        return 0
    
    # Case: 'a' has pre-release, 'b' does not -> 'a' is lower
    if a_pre is not None and b_pre is None:
        return -1
        
    # Case: 'a' does not have pre-release, 'b' has one -> 'a' is higher
    if a_pre is None and b_pre is not None:
        return 1

    # Case: Both have pre-release. Compare dot-by-dot.
    a_idents = a_pre.split('.')
    b_idents = b_pre.split('.')

    # Compare identifiers
    min_len = min(len(a_idents), len(b_idents))
    for i in range(min_len):
        ident_a = a_idents[i]
        ident_b = b_idents[i]
        
        # Determine if identifiers are numeric
        # SemVer numeric identifiers must not include leading zeroes.
        # We check if the string consists entirely of digits.
        is_num_a = ident_a.isdigit()
        is_num_b = ident_b.isdigit()
        
        if is_num_a and is_num_b:
            # Both numeric: compare numerically
            val_a = int(ident_a)
            val_b = int(ident_b)
            if val_a < val_b:
                return -1
            if val_a > val_b:
                return 1
        elif is_num_a and not is_num_b:
            # Numeric identifiers always rank lower than alphanumeric ones
            return -1
        elif not is_num_a and is_num_b:
            # Alphanumeric identifiers rank higher than numeric ones
            return 1
        else:
            # Both alphanumeric: compare lexically in ASCII order
            if ident_a < ident_b:
                return -1
            if ident_a > ident_b:
                return 1
    
    # If all preceding identifiers are equal, the one with more fields ranks higher
    if len(a_idents) < len(b_idents):
        return -1
    if len(a_idents) > len(b_idents):
        return 1

    return 0

# Unit Tests
if __name__ == "__main__":
    test_cases = [
        # (a, b, expected_result)
        # 1. Basic numeric comparison
        ("1.0.0", "2.0.0", -1),
        
        # 2. Pre-release vs Release (Pre-release has lower precedence)
        ("1.0.0-alpha", "1.0.0", -1),
        
        # 3. Numeric vs Alphanumeric identifiers (Numeric < Alphanumeric)
        ("1.0.0-alpha.1", "1.0.0-alpha.beta", -1),
        
        # 4. Differing field counts (Shorter set < Longer set if prefixes match)
        ("1.0.0-alpha", "1.0.0-alpha.1", -1),
        
        # 5. Build metadata is ignored
        ("1.0.0+build1", "1.0.0+build2", 0),
        
        # 6. Alphanumeric comparison (Lexical order)
        ("1.0.0-alpha.beta", "1.0.0-beta", -1),
    ]

    print("Running Unit Tests...")
    all_passed = True
    for i, (ver_a, ver_b, expected) in enumerate(test_cases, 1):
        result = compare_versions(ver_a, ver_b)
        status = "PASS" if result == expected else "FAIL"
        if result != expected:
            all_passed = False
        print(f"Test {i}: compare_versions('{ver_a}', '{ver_b}') -> {result} (Expected: {expected}) [{status}]")

    if all_passed:
        print("All tests passed!")
    else:
        print("Some tests failed.")
```
This looks complete and follows all instructions.

One minor point: `isdigit()` returns `True` for unicode digits too in Python 3 (e.g. '²'). `int()` might fail or behave differently. However, in SemVer context, ASCII digits are implied. `isdigit()` is generally sufficient for this task, or `ident_a.isdecimal()`. `isdecimal()` is stricter for ASCII digits usually, but `isdigit()` is standard for "looks like a number". Given the constraints, `isdigit()` is fine.

I'll stick with `isdigit()`.

Final verification of the code structure.
The function is defined.
The tests are included.
Rules are followed.

One specific detail:
`1.0.0-alpha.1` vs `1.0.0-alpha.beta`
`a` has `alpha`, `1`.
`b` has `alpha`, `beta`.
`alpha` == `alpha`.
`1` is numeric. `beta` is not.
Rule: Numeric < Alphanumeric.
So `1` < `beta`.
Result: -1.
My code:
`is_num_a` = True, `is_num_b` = False.
`elif is_num_a and not is_num_b: return -1`.
Correct.

`1.0.0-alpha` vs `1.0.0-alpha.1`
`a` has `alpha`.
`b` has `alpha`, `1`.
`alpha` == `alpha`.
Loop finishes.
`len(a)` = 1, `len(b)` = 2.
`len(a) < len(b)`.
Return -1.
Correct.

`1.0.0-alpha` vs `1.0.0`
`a` has `alpha`.
`b` has None.
`a_pre` is not None, `b_pre` is None.
Return -1.
Correct.

`1.0.0+build1` vs `1.0.0+build2`
Parsed core: `1.0.0`. Pre: None.
Both None.
Return 0.
Correct.

`1.0.0-alpha.beta` vs `
