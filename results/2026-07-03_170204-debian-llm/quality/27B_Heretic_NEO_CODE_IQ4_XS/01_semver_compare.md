# 27B_Heretic_NEO_CODE_IQ4_XS — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here’s a concise, spec-compliant implementation along with 6 targeted unit tests:

```python
def compare_versions(a: str, b: str) -> int:
    def parse(v: str):
        # 1. Strip build metadata (after '+')
        if '+' in v:
            v = v.split('+', 1)[0]

        # 2. Split pre-release (after '-')
        if '-' in v:
            base, pre = v.split('-', 1)
            pre_ids = pre.split('.')
        else:
            base = v
            pre_ids = None

        # 3. Parse major.minor.patch
        major, minor, patch = map(int, base.split('.'))
        return major, minor, patch, pre_ids

    m1, n1, p1, pre1 = parse(a)
    m2, n2, p2, pre2 = parse(b)

    # Compare major, minor, patch
    if m1 != m2: return 1 if m1 > m2 else -1
    if n1 != n2: return 1 if n1 > n2 else -1
    if p1 != p2: return 1 if p1 > p2 else -1

    # Pre-release precedence rules
    if pre1 is None and pre2 is None:
        return 0
    if pre1 is not None and pre2 is None:
        return -1  # pre-release < release
    if pre1 is None and pre2 is not None:
        return 1

    # Compare pre-release identifiers dot-by-dot
    for id1, id2 in zip(pre1, pre2):
        num1 = id1.isdigit()
        num2 = id2.isdigit()

        if num1 and num2:
            v1, v2 = int(id1), int(id2)
            if v1 != v2:
                return 1 if v1 > v2 else -1
        elif num1 and not num2:
            return -1  # numeric < alphanumeric
        elif not num1 and num2:
            return 1
        else:
            # Both alphanumeric: ASCII lexical order
            if id1 != id2:
                return 1 if id1 > id2 else -1

    # All preceding identifiers equal: longer set ranks higher
    if len(pre1) != len(pre2):
        return 1 if len(pre1) > len(pre2) else -1

    return 0


# 6 unit tests covering the required scenarios
def test_compare_versions():
    # 1. Pre-release vs release (pre-release is lower)
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1

    # 2. Numeric vs alphanumeric identifiers
    # Numeric identifiers always rank lower than alphanumeric
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1

    # 3. Differing field counts
    # When all preceding fields are equal, the larger set ranks higher
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1

    # 4. Build metadata is ignored
    assert compare_versions("1.0.0+build1", "1.0.0+build2") == 0

    # 5. Basic major/minor/patch precedence
    assert compare_versions("1.2.3", "1.10.0") == -1

    # 6. Equal versions with complex pre-release and metadata
    assert compare_versions("2.0.0-rc.1+meta", "2.0.0-rc.1+other") == 0

    print("All 6 tests passed.")


if __name__ == "__main__":
    test_compare_versions()
```

You can run this script directly to verify correctness. If you’d like it adapted to a specific test framework (pytest, unittest, etc.), I can adjust it.
