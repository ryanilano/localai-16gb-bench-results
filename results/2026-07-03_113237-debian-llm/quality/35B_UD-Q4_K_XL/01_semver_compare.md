# 35B_UD-Q4_K_XL — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here's a complete, standards-compliant implementation along with the requested unit tests:

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse(v: str):
        # Strip build metadata (everything after '+')
        core, _, _ = v.partition('+')
        # Split into core version and pre-release
        if '-' in core:
            ver, _, pre = core.partition('-')
        else:
            ver, pre = core, None
            
        major, minor, patch = map(int, ver.split('.'))
        return major, minor, patch, pre

    ma, na, pa, prea = parse(a)
    mb, nb, pb, preb = parse(b)

    # 1. Compare major, minor, patch numerically
    for ca, cb in [(ma, mb), (na, nb), (pa, pb)]:
        if ca < cb:
            return -1
        if ca > cb:
            return 1

    # 2. Core versions are equal. Handle pre-release precedence.
    if prea is None and preb is None:
        return 0
    if prea is None:
        return 1  # No pre-release > has pre-release
    if preb is None:
        return -1 # Has pre-release < no pre-release

    # 3. Both have pre-release: compare dot-separated identifiers
    ids_a = prea.split('.')
    ids_b = preb.split('.')

    for id_a, id_b in zip(ids_a, ids_b):
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()

        if is_num_a and is_num_b:
            ia, ib = int(id_a), int(id_b)
            if ia < ib: return -1
            if ia > ib: return 1
        elif is_num_a:
            return -1  # Numeric identifiers always < alphanumeric
        elif is_num_b:
            return 1   # Alphanumeric identifiers always > numeric
        else:
            if id_a < id_b: return -1
            if id_a > id_b: return 1

    # 4. All compared identifiers are equal. Larger field set ranks higher.
    if len(ids_a) < len(ids_b):
        return -1
    if len(ids_a) > len(ids_b):
        return 1
        
    return 0


class TestCompareVersions(unittest.TestCase):
    def test_prerelease_vs_release(self):
        """Pre-release always has lower precedence than release."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-beta"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        """Numeric identifiers always have lower precedence than alphanumeric ones."""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-beta", "1.0.0-1"), 1)

    def test_differing_field_counts(self):
        """Larger set of fields ranks higher when all preceding identifiers are equal."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

    def test_build_metadata_ignored(self):
        """Build metadata (after '+') is completely ignored in precedence."""
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+build1", "1.0.0-alpha+build2"), 0)
        self.assertEqual(compare_versions("1.0.0+20230101", "1.0.0+20230201"), 0)

    def test_core_version_comparison(self):
        """Standard numeric comparison for major.minor.patch."""
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        self.assertEqual(compare_versions("1.2.3", "1.2.4"), -1)
        self.assertEqual(compare_versions("2.0.0", "1.9.9"), 1)

    def test_lexical_alphanumeric_comparison(self):
        """Alphanumeric identifiers are compared lexically in ASCII order."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-beta"), -1)
        self.assertEqual(compare_versions("1.0.0-zeta", "1.0.0-alpha"), 1)


if __name__ == '__main__':
    unittest.main()
```

### Key Implementation Details:
1. **Parsing**: Uses `partition` to cleanly separate build metadata (`+`) and pre-release (`-`) without affecting hyphens inside pre-release identifiers.
2. **Core Comparison**: Iterates through major, minor, and patch pairs for clean, short-circuiting numeric comparison.
3. **Pre-release Logic**: 
   - Explicitly handles the `None` (no pre-release) cases first, satisfying the spec that `1.0.0 > 1.0.0-alpha`.
   - Uses `zip` to iterate dot-by-dot. Stops automatically when the shorter list ends.
   - Checks `isdigit()` to determine identifier type, applying the correct comparison rule (numeric vs numeric, alphanumeric vs alphanumeric, or numeric < alphanumeric).
4. **Field Count Rule**: After the `zip` loop, compares list lengths to satisfy "larger set of fields ranks higher".
5. **Build Metadata**: Stripped immediately during parsing, ensuring it never influences the return value.
