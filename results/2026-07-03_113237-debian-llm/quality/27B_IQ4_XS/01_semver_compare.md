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

Here's a complete, production-ready implementation following the SemVer 2.0.0 specification exactly, along with 6 targeted unit tests.

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    # 1. Ignore build metadata (everything after '+')
    a = a.split('+')[0]
    b = b.split('+')[0]
    
    # 2. Split into core version and optional pre-release
    a_parts = a.split('-', 1)
    b_parts = b.split('-', 1)
    
    # 3. Compare major, minor, patch numerically
    a_core = list(map(int, a_parts[0].split('.')))
    b_core = list(map(int, b_parts[0].split('.')))
    
    for x, y in zip(a_core, b_core):
        if x < y: return -1
        if x > y: return 1
        
    # 4. Handle pre-release precedence
    a_pre = a_parts[1] if len(a_parts) > 1 else None
    b_pre = b_parts[1] if len(b_parts) > 1 else None
    
    # Both are release versions -> equal
    if a_pre is None and b_pre is None:
        return 0
    # Pre-release has lower precedence than release
    if a_pre is None:
        return 1
    if b_pre is None:
        return -1
        
    # 5. Compare pre-release identifiers dot-by-dot
    a_ids = a_pre.split('.')
    b_ids = b_pre.split('.')
    
    for x, y in zip(a_ids, b_ids):
        x_is_num = x.isdigit()
        y_is_num = y.isdigit()
        
        if x_is_num and y_is_num:
            # Numeric identifiers compared numerically
            if int(x) < int(y): return -1
            if int(x) > int(y): return 1
        elif x_is_num and not y_is_num:
            # Numeric identifiers always rank lower than alphanumeric
            return -1
        elif not x_is_num and y_is_num:
            return 1
        else:
            # Alphanumeric identifiers compared lexically in ASCII order
            if x < y: return -1
            if x > y: return 1
            
    # 6. If all preceding identifiers are equal, larger set ranks higher
    if len(a_ids) < len(b_ids): return -1
    if len(a_ids) > len(b_ids): return 1
    
    return 0


# ================= UNIT TESTS =================
class TestCompareVersions(unittest.TestCase):
    
    def test_pre_release_vs_release(self):
        """Pre-release versions have lower precedence than the same version without one."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-beta"), 1)
        
    def test_numeric_vs_alphanumeric_identifiers(self):
        """Numeric identifiers rank lower than alphanumeric ones."""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.beta", "1.0.0-alpha.1"), 1)
        
    def test_differing_field_counts(self):
        """A larger set of pre-release fields ranks higher when all preceding ones are equal."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha"), 1)
        
    def test_build_metadata_ignored(self):
        """Build metadata must not be considered in precedence."""
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+metadata", "1.0.0-alpha"), 0)
        
    def test_core_version_comparison(self):
        """Major, minor, and patch are compared numerically."""
        self.assertEqual(compare_versions("1.0.0", "2.0.0"), -1)
        self.assertEqual(compare_versions("1.1.0", "1.2.0"), -1)
        self.assertEqual(compare_versions("1.0.1", "1.0.2"), -1)
        
    def test_equal_versions(self):
        """Identical versions (with or without metadata/pre-release) return 0."""
        self.assertEqual(compare_versions("1.0.0", "1.0.0"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha"), 0)
        self.assertEqual(compare_versions("1.0.0+meta", "1.0.0"), 0)

if __name__ == '__main__':
    unittest.main()
```

### Key Implementation Details:
1. **Build Metadata**: Stripped immediately using `.split('+')[0]` so it never influences comparison.
2. **Core Version**: Parsed into integers and compared left-to-right. First difference dictates the result.
3. **Pre-release vs Release**: Explicitly handled before identifier comparison. A version with `-` is strictly lower than the same version without it.
4. **Identifier Comparison**: 
   - `isdigit()` safely distinguishes numeric vs alphanumeric per SemVer rules.
   - Numeric pairs use `int()` for magnitude comparison.
   - Alphanumeric pairs use Python's native string comparison, which matches ASCII lexical order.
   - Mixed types default to numeric < alphanumeric as specified.
5. **Field Count Rule**: After exhausting the shorter identifier list, the longer list wins if all prior fields matched exactly.
