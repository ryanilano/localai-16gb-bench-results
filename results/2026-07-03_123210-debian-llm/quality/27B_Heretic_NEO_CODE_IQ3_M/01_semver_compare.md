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

Here’s a clean, spec-compliant implementation with 6 focused unit tests:

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compare two SemVer 2.0.0 version strings.
    Returns:
        -1 if a < b
         0 if a == b
         1 if a > b
    """
    def parse(version: str):
        # Ignore build metadata (after '+')
        core_and_pre = version.split('+')[0]

        # Split core and pre-release
        if '-' in core_and_pre:
            core, pre = core_and_pre.split('-', 1)
        else:
            core, pre = core_and_pre, None

        major, minor, patch = map(int, core.split('.'))
        return major, minor, patch, pre

    maj_a, min_a, pat_a, pre_a = parse(a)
    maj_b, min_b, pat_b, pre_b = parse(b)

    # 1. Compare major, minor, patch numerically
    if (maj_a, min_a, pat_a) != (maj_b, min_b, pat_b):
        return -1 if (maj_a, min_a, pat_a) < (maj_b, min_b, pat_b) else 1

    # Same core version: handle pre-release precedence
    # No pre-release on either → equal
    if pre_a is None and pre_b is None:
        return 0
    # Release > pre-release
    if pre_a is None:
        return 1
    if pre_b is None:
        return -1

    # Both have pre-release: compare dot-separated identifiers
    ids_a = pre_a.split('.')
    ids_b = pre_b.split('.')

    for id_a, id_b in zip(ids_a, ids_b):
        a_is_num = id_a.isdigit()
        b_is_num = id_b.isdigit()

        # Numeric vs numeric: compare as integers
        if a_is_num and b_is_num:
            if int(id_a) != int(id_b):
                return -1 if int(id_a) < int(id_b) else 1
        # Numeric always ranks lower than alphanumeric
        elif a_is_num and not b_is_num:
            return -1
        elif not a_is_num and b_is_num:
            return 1
        # Both alphanumeric: ASCII lexical comparison
        else:
            if id_a != id_b:
                return -1 if id_a < id_b else 1

    # All compared identifiers are equal; longer set wins
    if len(ids_a) != len(ids_b):
        return -1 if len(ids_a) < len(ids_b) else 1

    return 0


# =====================
# 6 UNIT TESTS
# =====================
if __name__ == "__main__":
    # 1. Pre-release vs release: 1.0.0-alpha < 1.0.0
    assert compare_versions("1.0.0-alpha", "1.0.0") == -1, "Test 1 failed"

    # 2. Release vs pre-release: 1.0.0 > 1.0.0-beta
    assert compare_versions("1.0.0", "1.0.0-beta") == 1, "Test 2 failed"

    # 3. Numeric vs alphanumeric identifiers:
    #    1.0.0-alpha.1 < 1.0.0-alpha.beta (numeric < alnum)
    assert compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta") == -1, "Test 3 failed
