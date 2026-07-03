# 27B_HauhauCS_Balanced_Q3_K_P — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here's a complete, spec-compliant implementation with 6 targeted unit tests:

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if a == b, 1 if a > b.
    """
    def parse(v: str):
        # Strip build metadata (ignored in precedence)
        if '+' in v:
            v = v.split('+', 1)[0]
        # Extract pre-release tag
        pre = None
        if '-' in v:
            v, pre = v.split('-', 1)
        major, minor, patch = map(int, v.split('.'))
        pre_list = pre.split('.') if pre else []
        return major, minor, patch, pre_list

    a_maj, a_min, a_pat, a_pre = parse(a)
    b_maj, b_min, b_pat, b_pre = parse(b)

    # 1. Compare major, minor, patch numerically
    for x, y in [(a_maj, b_maj), (a_min, b_min), (a_pat, b_pat)]:
        if x < y: return -1
        if x > y: return 1

    # 2. Compare pre-release presence
    a_has_pre = len(a_pre) > 0
    b_has_pre = len(b_pre) > 0

    if not a_has_pre and not b_has_pre:
        return 0
    if a_has_pre and not b_has_pre:
        return -1  # Pre-release < Release
    if not a_has_pre and b_has_pre:
        return 1   # Release > Pre-release

    # 3. Compare pre-release identifiers dot-by-dot
    min_len = min(len(a_pre), len(b_pre))
    for i in range(min_len):
        id_a = a_pre[i]
        id_b = b_pre[i]

        # SemVer numeric identifiers comprise only ASCII digits [0-9]
        a_is_num = id_a.isdigit()
        b_is_num = id_b.isdigit()

        if a_is_num and b_is_num:
            if int(id_a) < int(id_b): return -1
            if int(id_a) > int(id_b): return 1
        elif a_is_num and not b_is_num:
            return -1  # Numeric < Alphanumeric
        elif not a_is_num and b_is_num:
            return 1   # Alphanumeric > Numeric
        else:
            # Both alphanumeric: lexical ASCII comparison
            if id_a < id_b: return -1
            if id_a > id_b: return 1

    # 4. Larger set of fields ranks higher when all preceding are equal
    if len(a_pre) < len(b_pre): return -1
    if len(a_pre) > len(b_pre): return 1
    return 0


class TestCompareVersions(unittest.TestCase):
    def test_prerelease_vs_release(self):
        """A version WITH a pre-release tag has LOWER precedence than the same version without one."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

    def test_numeric_vs_alphanumeric(self):
        """Numeric identifiers always rank lower than alphanumeric ones."""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)

    def test_differing_field_counts(self):
        """A larger set of pre-release fields ranks higher when all preceding ones are equal."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)

    def test_build_metadata_ignored(self):
        """Build metadata (after '+') is IGNORED in precedence."""
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+meta", "1.0.0-alpha"), 0)

    def test_equal_versions(self):
        """Identical versions return 0."""
        self.assertEqual(compare_versions("1.2.3", "1.2.3"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha.1+build", "1.0.0-alpha.1+other"), 0)

    def test_major_minor_patch_comparison(self):
        """Major, minor, and patch are compared numerically left-to-right."""
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        self.assertEqual(compare_versions("1.1.0", "1.0.0"), 1)
        self.assertEqual(compare_versions("1.0.1", "1.0.0"), 1)


if __name__ == '__main__':
    unittest.main()
```

### Key Implementation Details:
1. **Parsing**: Efficiently strips build metadata and pre-release tags using single splits, avoiding regex overhead while remaining fully compliant.
2. **Precedence Logic**: Follows SemVer 2.0.0 Section
