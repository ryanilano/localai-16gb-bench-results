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

Here's a complete, production-ready implementation following the Semantic Versioning 2.0.0 specification exactly, along with 6 targeted unit tests.

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if a == b, 1 if a > b.
    """
    def parse(v: str):
        # Build metadata (after '+') is ignored in precedence
        v = v.split('+')[0]
        
        # Separate core version and pre-release
        if '-' in v:
            core, pre = v.split('-', 1)
        else:
            core, pre = v, None
            
        major, minor, patch = map(int, core.split('.'))
        pre_ids = pre.split('.') if pre else None
        return major, minor, patch, pre_ids

    ma, mi, pa, pra = parse(a)
    mb, mib, pb, prb = parse(b)

    # 1. Compare major, minor, patch numerically
    for x, y in [(ma, mb), (mi, mib), (pa, pb)]:
        if x < y:
            return -1
        if x > y:
            return 1

    # 2. Core versions are equal. Handle pre-release presence
    if pra is None and prb is None:
        return 0
    if pra is None:
        return 1  # a is release, b is pre-release -> a > b
    if prb is None:
        return -1 # a is pre-release, b is release -> a < b

    # 3. Compare pre-release identifiers dot-by-dot
    min_len = min(len(pra), len(prb))
    for i in range(min_len):
        id_a, id_b = pra[i], prb[i]
        a_is_num = id_a.isdigit()
        b_is_num = id_b.isdigit()

        if a_is_num and b_is_num:
            # Numeric identifiers compared as integers
            va, vb = int(id_a), int(id_b)
            if va < vb: return -1
            if va > vb: return 1
        elif a_is_num and not b_is_num:
            # Numeric identifiers always rank lower than alphanumeric
            return -1
        elif not a_is_num and b_is_num:
            return 1
        else:
            # Alphanumeric identifiers compared lexically in ASCII order
            if id_a < id_b: return -1
            if id_a > id_b: return 1

    # 4. All compared fields are equal. Larger set of fields ranks higher.
    if len(pra) < len(prb): return -1
    if len(pra) > len(prb): return 1
    return 0


# ------------------------------------------------------------------
# Unit Tests
# ------------------------------------------------------------------
def run_tests():
    # 1. Pre-release vs release: release has higher precedence
    assert compare_versions("1.0.0", "1.0.0-alpha") == 1, "Test 1 failed: pre-release vs release"
    
    # 2. Numeric vs alphanumeric identifiers: numeric ranks lower
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Test 2 failed: numeric vs alphanumeric"
    
    # 3. Differing field counts: more fields rank higher when preceding are equal
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1, "Test 3 failed: differing field counts"
    
    # 4. Build metadata is completely ignored in precedence
    assert compare_versions("1.0.0+build.123", "1.0.0+build.456") == 0, "Test 4 failed: build metadata ignored"
    
    # 5. Major/Minor/Patch numeric comparison
    assert compare_versions("2.1.0", "2.0.9") == 1, "Test 5 failed: major/minor/patch comparison"
    
    # 6. Alphanumeric lexical ASCII ordering
    assert compare_versions("1.0.0-alpha", "1.0.0-beta") == -1, "Test 6 failed: alphanumeric lexical order"
    
    print("✅ All 6 unit tests passed successfully.")

if __name__ == "__main__":
    run_tests()
```

### Key Implementation Details:
1. **Parsing**: Splits on `+` first to strip build metadata per spec. Then splits on `-` to isolate pre-release tags. Core version is split by `.` and cast to `int`.
2. **Precedence Logic**: 
   - Core version comparison happens first.
   - If cores match, presence of a pre-release tag dictates precedence (release > pre-release).
   - Identifier comparison strictly follows SemVer 2.0.0 §11: numeric vs numeric (int comparison), numeric vs alphanumeric (numeric wins lower), alphanumeric vs alphanumeric (ASCII lexicographical).
   - Length tie-breaker applied only when all compared identifiers are identical.
3. **Complexity**: `O(N)` where `N` is the number of identifiers in the longer pre-release string. Space complexity is `O(N)` for splitting strings.
