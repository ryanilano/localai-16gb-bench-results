# 27B_IQ4_XS — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here's a complete implementation following the Semantic Versioning 2.0.0 specification exactly, along with 6 targeted unit tests.

```python
def compare_versions(a: str, b: str) -> int:
    # 1. Ignore build metadata (everything after '+')
    a_main = a.split('+')[0]
    b_main = b.split('+')[0]
    
    # 2. Separate core version from pre-release tag
    a_ver_str, _, a_pre = a_main.partition('-')
    b_ver_str, _, b_pre = b_main.partition('-')
    
    # 3. Compare major, minor, patch numerically
    a_ver = list(map(int, a_ver_str.split('.')))
    b_ver = list(map(int, b_ver_str.split('.')))
    for va, vb in zip(a_ver, b_ver):
        if va < vb: return -1
        if va > vb: return 1
        
    # 4. Handle pre-release precedence
    # No pre-release on both -> equal
    if not a_pre and not b_pre:
        return 0
    # Release has higher precedence than pre-release
    if not a_pre:
        return 1
    if not b_pre:
        return -1
        
    # 5. Compare pre-release identifiers dot-by-dot
    a_ids = a_pre.split('.')
    b_ids = b_pre.split('.')
    
    for i in range(max(len(a_ids), len(b_ids))):
        # Larger set of fields ranks higher when all preceding are equal
        if i >= len(a_ids):
            return -1
        if i >= len(b_ids):
            return 1
            
        ai, bi = a_ids[i], b_ids[i]
        a_is_num = ai.isdigit()
        b_is_num = bi.isdigit()
        
        if a_is_num and b_is_num:
            # Numeric identifiers compared as integers
            va, vb = int(ai), int(bi)
            if va < vb: return -1
            if va > vb: return 1
        elif a_is_num:
            # Numeric identifiers always rank lower than alphanumeric
            return -1
        elif b_is_num:
            return 1
        else:
            # Alphanumeric identifiers compared lexically in ASCII order
            if ai < bi: return -1
            if ai > bi: return 1
            
    return 0

# ================= UNIT TESTS =================
if __name__ == "__main__":
    # 1. Pre-release vs release: pre-release has lower precedence
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1, "Test 1 failed: pre-release vs release"
    
    # 2. Numeric vs alphanumeric identifiers: numeric ranks lower
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Test 2 failed: numeric vs alphanumeric"
    
    # 3. Differing field counts: larger set of fields ranks higher
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1, "Test 3 failed: differing field counts"
    
    # 4. Build metadata is completely ignored in precedence
    assert compare_versions("1.0.0+build1", "1.0.0+build2") == 0, "Test 4 failed: build metadata ignored"
    
    # 5. Standard major/minor/patch numeric comparison
    assert compare_versions("2.1.0", "2.0.9") == 1, "Test 5 failed: major/minor/patch precedence"
    
    # 6. Equal versions with mixed metadata & pre-release
    assert compare_versions("1.2.3-alpha.1+build", "1.2.3-alpha.1+other") == 0, "Test 6 failed: equal versions"
    
    print("All 6 unit tests passed.")
```

### Key Design Choices:
- **`partition('-')`** safely handles versions without a pre-release tag without raising index errors.
- **`isdigit()`** correctly distinguishes numeric identifiers from alphanumeric ones per the SemVer spec.
- **ASCII lexical comparison** leverages Python's native string comparison, which matches ASCII ordering for standard SemVer characters.
- **Field count logic** explicitly handles the rule that `1.0.0-alpha < 1.0.0-alpha.1` by checking index bounds during iteration.
- Build metadata is stripped immediately, ensuring it never influences the comparison logic.
