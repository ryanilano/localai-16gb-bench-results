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

Here's a clean, spec-compliant implementation with the requested unit tests:

```python
def compare_versions(a: str, b: str) -> int:
    # Strip build metadata (everything after '+')
    a_core = a.split('+', 1)[0]
    b_core = b.split('+', 1)[0]

    # Split core version and pre-release
    if '-' in a_core:
        a_ver, a_pre = a_core.split('-', 1)
        a_pre_ids = a_pre.split('.')
    else:
        a_ver = a_core
        a_pre_ids = None

    if '-' in b_core:
        b_ver, b_pre = b_core.split('-', 1)
        b_pre_ids = b_pre.split('.')
    else:
        b_ver = b_core
        b_pre_ids = None

    # Compare major.minor.patch numerically
    a_parts = list(map(int, a_ver.split('.')))
    b_parts = list(map(int, b_ver.split('.')))

    for av, bv in zip(a_parts, b_parts):
        if av < bv: return -1
        if av > bv: return 1

    # Core versions equal: handle pre-release precedence
    if a_pre_ids is None and b_pre_ids is None:
        return 0
    if a_pre_ids is None:
        return 1   # release > pre-release
    if b_pre_ids is None:
        return -1  # pre-release < release

    # Compare pre-release identifiers dot-by-dot
    for id_a, id_b in zip(a_pre_ids, b_pre_ids):
        a_num = id_a.isdigit()
        b_num = id_b.isdigit()

        if a_num and b_num:
            av, bv = int(id_a), int(id_b)
            if av < bv: return -1
            if av > bv: return 1
        elif a_num and not b_num:
            return -1  # numeric < alphanumeric
        elif not a_num and b_num:
            return 1   # alphanumeric > numeric
        else:
            # both alphanumeric: lexical ASCII comparison
            if id_a < id_b: return -1
            if id_a > id_b: return 1

    # All preceding identifiers equal; longer set has higher precedence
    if len(a_pre_ids) < len(b_pre_ids): return -1
    if len(a_pre_ids) > len(b_pre_ids): return 1
    return 0


if __name__ == "__main__":
    # 1. Pre-release vs release
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1

    # 2. Numeric vs alphanumeric identifiers
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1

    # 3. Differing field counts
    assert compare_versions("1.0.0-alpha", "1.0.0-alpha.1") == -1

    # 4. Build metadata is ignored
    assert compare_versions("1.0.0+build.1", "1.0.0+build.2") == 0

    # 5. Major/minor/patch numeric comparison
    assert compare_versions("2.1.0", "2.0.0") == 1

    # 6. Numeric pre-release identifiers compared as numbers (not lexically)
    assert compare_versions("1.0.0-rc.1", "1.0.0-rc.10") == -1

    print("All 6 tests passed.")
```

This follows the SemVer 2.0.0 precedence rules exactly and runs in O(N) time where N is the length of the version strings. Let me know if you need it integrated into a test framework or extended with validation.
