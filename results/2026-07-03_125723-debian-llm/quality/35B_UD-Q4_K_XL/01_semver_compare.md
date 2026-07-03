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

Here's a complete, well-documented implementation following the SemVer 2.0.0 precedence rules exactly, along with 6 targeted unit tests.

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compare two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if a == b, 1 if a > b.
    """
    def parse(v: str):
        # Build metadata (after '+') is ignored in precedence
        core_pre = v.split('+')[0]
        # Split core version and pre-release tag
        parts = core_pre.split('-', 1)
        core_str = parts[0]
        pre_str = parts[1] if len(parts) > 1 else None
        
        major, minor, patch = map(int, core_str.split('.'))
        return major, minor, patch, pre_str

    ma, na, pa, prea = parse(a)
    mb, nb, pb, preb = parse(b)

    # 1. Compare major, minor, patch numerically
    for ca, cb in [(ma, mb), (na, nb), (pa, pb)]:
        if ca < cb: return -1
        if ca > cb: return 1

    # 2. Compare pre-release identifiers
    if prea is None and preb is None:
        return 0
    if prea is None:
        return 1  # Release > Pre-release
    if preb is None:
        return -1 # Pre-release < Release

    ids_a = prea.split('.')
    ids_b = preb.split('.')

    for ida, idb in zip(ids_a, ids_b):
        is_num_a = ida.isdigit()
        is_num_b = idb.isdigit()

        if is_num_a and is_num_b:
            # Both numeric: compare as integers
            if int(ida) < int(idb): return -1
            if int(ida) > int(idb): return 1
        elif is_num_a:
            return -1  # Numeric identifiers always rank lower than alphanumeric
        elif is_num_b:
            return 1   # Alphanumeric identifiers rank higher than numeric
        else:
            # Both alphanumeric: compare lexically in ASCII order
            if ida < idb: return -1
            if ida > idb: return 1

    # 3. Larger set of fields has higher precedence when all preceding ones are equal
    if len(ids_a) < len(ids_b): return -1
    if len(ids_a) > len(ids_b): return 1

    return 0


# ==========================
# Unit Tests
# ==========================
import unittest

class TestSemVerComparison(unittest.TestCase):
    
    def test_pre_release_vs_release(self):
        """A version with a pre-release tag has LOWER precedence than the same version without one."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-alpha"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        """Numeric identifiers always rank lower than alphanumeric ones."""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1"), 1)

    def test_differing_field_counts(self):
        """A larger set of fields ranks higher when all preceding ones are equal."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)

    def test_build_metadata_ignored(self):
        """Build metadata (after '+') is completely ignored in precedence."""
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+build1", "1.0.0-alpha+build2"), 0)

    def test_core_version_comparison(self):
        """Major, minor, and patch are compared numerically."""
        self.assertEqual(compare_versions("2.0.0", "1.9.9"), 1)
        self.assertEqual(compare_versions("1.0.0", "1.1.0"), -1)
        self.assertEqual(compare_versions("0.9.9", "0.10.0"), -1)

    def test_equal_versions(self):
        """Identical versions return 0."""
        self.assertEqual(compare_versions("1.2.3", "1.2.3"), 0)
        self.assertEqual(compare_versions("1.2.3-alpha.1", "1.2.3-alpha.1"), 0)

if __name__ == "__main__":
    unittest.main()
```

### Key Implementation Details:
1. **Metadata Ignored**: Splits on `+` immediately, discarding anything after it.
2. **Pre-release vs Release**: Explicitly checks for `None` pre-release strings. Missing pre-release always wins.
3. **Identifier Comparison**: Uses `zip()` to iterate dot-separated identifiers left-to-right. Handles numeric vs alphanumeric mixing exactly per spec.
4. **Field Count Rule**: After the loop, compares list lengths. Longer pre-release = higher precedence if prefixes match.
5. **Type Safety & Clarity**: Uses clean parsing helper and explicit return values matching the contract.
