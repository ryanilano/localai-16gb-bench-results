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

Here's the complete implementation following Semantic Versioning 2.0.0 precedence rules, along with the requested unit tests:

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two SemVer 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse(v: str):
        # Build metadata (after '+') is ignored for precedence
        v = v.split('+')[0]
        
        # Separate pre-release identifier
        if '-' in v:
            base, pre = v.split('-', 1)
        else:
            base, pre = v, None
            
        parts = base.split('.')
        major, minor, patch = int(parts[0]), int(parts[1]), int(parts[2])
        pre_ids = pre.split('.') if pre else None
        return major, minor, patch, pre_ids

    a_maj, a_min, a_pat, a_pre = parse(a)
    b_maj, b_min, b_pat, b_pre = parse(b)

    # 1. Compare major.minor.patch numerically
    if (a_maj, a_min, a_pat) != (b_maj, b_min, b_pat):
        return -1 if (a_maj, a_min, a_pat) < (b_maj, b_min, b_pat) else 1

    # 2. Handle pre-release presence
    if a_pre is None and b_pre is None:
        return 0
    if a_pre is not None and b_pre is None:
        return -1  # Pre-release has lower precedence than release
    if a_pre is None and b_pre is not None:
        return 1

    # 3. Compare pre-release identifiers dot-by-dot
    for id_a, id_b in zip(a_pre, b_pre):
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()

        if is_num_a and is_num_b:
            # Both numeric: compare as integers
            val_a, val_b = int(id_a), int(id_b)
            if val_a != val_b:
                return -1 if val_a < val_b else 1
        elif is_num_a:
            # Numeric always ranks lower than alphanumeric
            return -1
        elif is_num_b:
            return 1
        else:
            # Both alphanumeric: compare lexically in ASCII order
            if id_a != id_b:
                return -1 if id_a < id_b else 1

    # 4. If one identifier list is a prefix of the other, 
    # the one with more fields has higher precedence
    if len(a_pre) != len(b_pre):
        return -1 if len(a_pre) < len(b_pre) else 1

    return 0


# ================= UNIT TESTS =================
if __name__ == "__main__":
    # 1. Pre-release vs release: release has higher precedence
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1, "Test 1 Failed: Pre-release vs release"
    
    # 2. Numeric vs alphanumeric identifiers: numeric ranks lower
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Test 2 Failed: Numeric vs alphanumeric identifiers"
    
    # 3. Differing field counts: larger set ranks higher when preceding fields match
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1, "Test 3 Failed: Differing field counts"
    
    # 4. Build metadata is completely ignored
    assert compare_versions("1.0.0+build1", "1.0.0+build2") == 0, "Test 4 Failed: Build metadata ignored"
    
    # 5. Basic numeric comparison (patch level)
    assert compare_versions("1.2.3", "1.2.4") == -1, "Test 5 Failed: Basic patch comparison"
    
    # 6. Lexical comparison of alphanumeric identifiers
    assert compare_versions("1.0.0-alpha", "1.0.0-beta") == -1, "Test 6 Failed: Lexical alphanumeric comparison"
    
    print("All 6 unit tests passed successfully.")
```

### Key Implementation Details:
1. **Parsing & Build Metadata**: Splits off `+...` immediately since build metadata has zero effect on precedence.
2. **Core Version Comparison**: Compares `(major, minor, patch)` tuples numerically first.
3. **Pre-release Presence**: A version with a pre-release tag is always less than the same version without one.
4. **Dot-separated Identifier Logic**: 
   - Uses `isdigit()` to distinguish numeric vs alphanumeric identifiers.
   - Numeric identifiers are cast to `int` for comparison.
   - Mixed types default to numeric `< alphanumeric`.
   - Pure alphanumeric strings use Python's default lexicographical comparison (ASCII order).
   - Handles differing identifier lengths by comparing lengths after the loop exhausts the shorter list.
