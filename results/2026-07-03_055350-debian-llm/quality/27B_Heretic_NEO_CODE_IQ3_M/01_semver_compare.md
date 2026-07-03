# 27B_Heretic_NEO_CODE_IQ3_M — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here’s a clean, spec-compliant implementation with 6 targeted unit tests:

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compare two SemVer 2.0.0 strings.
    Returns:
        -1 if a < b
         0 if a == b
         1 if a > b
    """

    def parse(v: str):
        # Build metadata (after '+') is ignored for precedence
        if '+' in v:
            v = v.split('+', 1)[0]

        # Split pre-release if present
        if '-' in v:
            version, pre = v.split('-', 1)
        else:
            version, pre = v, None

        major, minor, patch = map(int, version.split('.'))
        return (major, minor, patch), pre

    a_ver, a_pre = parse(a)
    b_ver, b_pre = parse(b)

    # 1. Compare major.minor.patch numerically
    if a_ver != b_ver:
        return -1 if a_ver < b_ver else 1

    # Base versions are equal
    # 2. A version with a pre-release has LOWER precedence than the same version without
    if a_pre is None and b_pre is None:
        return 0
    if a_pre is not None and b_pre is None:
        return -1
    if a_pre is None and b_pre is not None:
        return 1

    # 3. Both have pre-release: compare dot-separated identifiers left to right
    a_ids = a_pre.split('.')
    b_ids = b_pre.split('.')

    for x, y in zip(a_ids, b_ids):
        if x == y:
            continue

        x_num = x.isdigit()
        y_num = y.isdigit()

        if x_num and y_num:
            # Numeric identifiers compared numerically
            if int(x) < int(y):
                return -1
            if int(x) > int(y):
                return 1
        elif x_num and not y_num:
            # Numeric < alphanumeric
            return -1
        elif not x_num and y_num:
            return 1
        else:
            # Both alphanumeric: lexical (ASCII) comparison
            if x < y:
                return -1
            if x > y:
                return 1

    # 4. If all preceding identifiers are equal, a larger set of fields ranks higher
    if len(a_ids) < len(b_ids):
        return -1
    if len(a_ids) > len(b_ids):
        return 1

    return 0


# 6 unit tests covering the required scenarios
if __name__ == "__main__":
    # 1. Basic major/minor/patch precedence
    assert compare_versions("1.2.3", "2.0.0") == -1, "Failed: basic version comparison"

    # 2. Pre-release vs release: pre-release has lower precedence
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1, "Failed: pre-release vs release"

    # 3. Numeric vs alphanumeric identifiers: numeric ranks lower
    # Example from prompt: 1.0.0-alpha.1 < 1.0.0-alpha.beta
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Failed: numeric vs alphanumeric"

    # 4. Differing field counts: shorter set ranks lower when preceding are equal
    # Example from prompt: 1.0.0-alpha < 1.0.0-alpha.1
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1, "Failed: differing field counts"

    # 5. Build metadata is ignored
    assert compare_versions("1.0.0+build.1", "1.0.0+build.2") == 0, "Failed: build metadata ignored"

    # 6. Numeric pre-release identifiers compared numerically (not lexically)
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.2") == -1, "Failed: numeric pre-release comparison"

    print("All tests passed.")
```

This implementation:
- Strictly follows SemVer 2.0.0 precedence rules.
- Handles build metadata, pre-release vs release, numeric vs alphanumeric IDs, and field count rules exactly as specified.
- Includes 6 focused unit tests matching your requested scenarios.
