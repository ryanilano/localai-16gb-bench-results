# 27B_NEO_CODE_IQ4_XS — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here's a complete, robust implementation following the SemVer 2.0.0 specification exactly, along with 6 targeted unit tests.

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if a == b, 1 if a > b.
    """
    def parse(v: str):
        # Ignore build metadata (everything after '+')
        core = v.split('+')[0]
        
        # Separate pre-release if present
        if '-' in core:
            main, pre = core.split('-', 1)
        else:
            main, pre = core, None
            
        # Parse major.minor.patch
        parts = main.split('.')
        major, minor, patch = int(parts[0]), int(parts[1]), int(parts[2])
        
        # Split pre-release identifiers
        pre_ids = pre.split('.') if pre else None
        return major, minor, patch, pre_ids

    a_maj, a_min, a_pat, a_pre = parse(a)
    b_maj, b_min, b_pat, b_pre = parse(b)

    # 1. Compare major, minor, patch numerically
    if (a_maj, a_min, a_pat) < (b_maj, b_min, b_pat):
        return -1
    if (a_maj, a_min, a_pat) > (b_maj, b_min, b_pat):
        return 1

    # 2. Compare pre-release presence
    if a_pre is None and b_pre is None:
        return 0
    if a_pre is None:
        return 1  # a has no pre-release, so a > b
    if b_pre is None:
        return -1 # b has no pre-release, so a < b

    # 3. Compare pre-release identifiers dot-by-dot
    for i in range(min(len(a_pre), len(b_pre))):
        a_id, b_id = a_pre[i], b_pre[i]
        a_is_num = a_id.isdecimal()
        b_is_num = b_id.isdecimal()

        if a_is_num and b_is_num:
            # Both numeric: compare as integers
            a_val, b_val = int(a_id), int(b_id)
        elif a_is_num:
            # Numeric always ranks lower than alphanumeric
            a_val, b_val = -1, 1
        elif b_is_num:
            a_val, b_val = 1, -1
        else:
            # Both alphanumeric: compare lexically in ASCII order
            a_val, b_val = a_id, b_id

        if a_val < b_val:
            return -1
        if a_val > b_val:
            return 1

    # 4. All compared identifiers are equal; larger set ranks higher
    if len(a_pre) < len(b_pre):
        return -1
    elif len(a_pre) > len(b_pre):
        return 1
    return 0


# ==========================
# UNIT TESTS
# ==========================
def run_tests():
    # 1. Pre-release vs release
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1, "Failed: pre-release vs release"
    
    # 2. Numeric vs alphanumeric identifiers
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Failed: numeric vs alphanumeric"
    
    # 3. Differing field counts
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1, "Failed: differing field counts"
    
    # 4. Build metadata ignored
    assert compare_versions("1.0.0+build123", "1.0.0+build456") == 0, "Failed: build metadata ignored"
    
    # 5. Major/Minor/Patch precedence
    assert compare_versions("2.0.0", "1.9.9") == 1, "Failed: major/minor/patch comparison"
    
    # 6. Equal versions
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha") == 0, "Failed: equal versions"
    
    print("✅ All 6 unit tests passed successfully!")

if __name__ == "__main__":
    run_tests()
```

### Key Implementation Details:
1. **Parsing & Metadata Ignorance**: Splits on `+` first to completely discard build metadata before any comparison logic runs.
2. **Core Version Comparison**: Compares `(major, minor, patch)` as a tuple for clean, Pythonic numeric precedence.
3. **Pre-release Rules**:
   - Handles the explicit rule that `version-with-prerelease < version-without-prerelease`.
   - Uses `.isdecimal()` to safely identify numeric identifiers (strictly ASCII digits).
   - Enforces `numeric < alphanumeric` by assigning sentinel values (`-1` vs `1`).
   - Falls back to standard string comparison for alphanumeric identifiers, which natively follows ASCII lexicographical order in Python.
   - Correctly implements the "larger set of fields ranks higher" rule after the dot-by-dot loop completes.
