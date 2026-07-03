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

Here's a complete implementation following Semantic Versioning 2.0.0 precedence rules, along with 6 targeted unit tests:

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compare two SemVer 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse(v: str):
        # Strip build metadata (everything after '+')
        v = v.split('+')[0]
        
        # Separate core version from pre-release
        if '-' in v:
            core, pre = v.split('-', 1)
            pre_list = pre.split('.')
        else:
            core, pre_list = v, None
            
        major, minor, patch = map(int, core.split('.'))
        return major, minor, patch, pre_list

    maj_a, min_a, pat_a, pre_a = parse(a)
    maj_b, min_b, pat_b, pre_b = parse(b)

    # 1. Compare major, minor, patch numerically
    if maj_a != maj_b:
        return (maj_a > maj_b) - (maj_a < maj_b)
    if min_a != min_b:
        return (min_a > min_b) - (min_a < min_b)
    if pat_a != pat_b:
        return (pat_a > pat_b) - (pat_a < pat_b)

    # 2. Compare pre-release presence
    # A version without a pre-release tag has HIGHER precedence
    if pre_a is None and pre_b is None:
        return 0
    if pre_a is not None and pre_b is None:
        return -1
    if pre_a is None and pre_b is not None:
        return 1

    # 3. Compare pre-release identifiers dot-by-dot
    min_len = min(len(pre_a), len(pre_b))
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b[i]
        
        # Check if identifiers are purely numeric (ASCII digits only)
        num_a = all(c in '0123456789' for c in id_a)
        num_b = all(c in '0123456789' for c in id_b)

        if num_a and num_b:
            # Numeric identifiers compared as integers
            int_a, int_b = int(id_a), int(id_b)
            if int_a != int_b:
                return (int_a > int_b) - (int_a < int_b)
        elif num_a and not num_b:
            # Numeric ranks lower than alphanumeric
            return -1
        elif not num_a and num_b:
            return 1
        else:
            # Alphanumeric compared lexically in ASCII order
            if id_a != id_b:
                return (id_a > id_b) - (id_a < id_b)

    # 4. Larger set of fields ranks higher when all preceding are equal
    return (len(pre_a) > len(pre_b)) - (len(pre_a) < len(pre_b))


# =============================================================================
# Unit Tests
# =============================================================================
class TestCompareVersions(unittest.TestCase):
    def test_prerelease_vs_release(self):
        """Pre-release has lower precedence than the same normal version"""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)

    def test_numeric_vs_alphanumeric(self):
        """Numeric identifiers always rank lower than alphanumeric ones"""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)

    def test_differing_field_counts(self):
        """A larger set of fields ranks higher when all preceding identifiers are equal"""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)

    def test_build_metadata_ignored(self):
        """Build metadata (after '+') is completely ignored in precedence"""
        self.assertEqual(compare_versions("1.0.0+build", "1.0.0+other"), 0)

    def test_equal_versions(self):
        """Identical versions return 0"""
        self.assertEqual(compare_versions("1.2.3", "1.2.3"), 0)

    def test_major_version_difference(self):
        """Major version difference determines precedence regardless of other fields"""
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)


if __name__ == '__main__':
    unittest.main()
```

### Key Implementation Details:
1. **Parsing**: Strips build metadata first, then splits on `-` to isolate pre-release tags. Core version is parsed into integers.
2. **Core Comparison**: Compares `major`, `minor`, `patch` sequentially. First difference determines the result.
3. **Pre-release Presence**: Explicitly handles the rule that `1.0.0 > 1.0.0-alpha`.
4. **Identifier Comparison**: 
   - Checks if an identifier is purely numeric using ASCII digit validation.
   - Numeric vs Numeric: compared as integers.
   - Numeric vs Alphanumeric: numeric always loses.
   - Alphanumeric vs Alphanumeric: compared using standard ASCII string comparison.
5. **Field Count**: If all compared identifiers are equal, the version with more identifiers wins.
6. **Return Trick**: `(a > b) - (a < b)` cleanly returns `1`, `-1`, or `0` without verbose `if/else` chains.
