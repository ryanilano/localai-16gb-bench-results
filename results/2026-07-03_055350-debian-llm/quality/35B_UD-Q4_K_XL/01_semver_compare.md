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

Here's a complete, production-ready implementation following the Semantic Versioning 2.0.0 specification, along with the requested unit tests.

```python
import re
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if a == b, 1 if a > b.
    """
    # Regex captures: MAJOR, MINOR, PATCH, pre-release, build metadata
    pattern = re.compile(r'^(\d+)\.(\d+)\.(\d+)(?:-([a-zA-Z0-9-]+))?(?:\+([a-zA-Z0-9-]+))?$')
    m_a = pattern.fullmatch(a)
    m_b = pattern.fullmatch(b)
    
    if not m_a or not m_b:
        raise ValueError("Invalid Semantic Version string")

    major_a, minor_a, patch_a, pre_a, _ = m_a.groups()
    major_b, minor_b, patch_b, pre_b, _ = m_b.groups()

    # 1. Compare major.minor.patch numerically
    core_a = (int(major_a), int(minor_a), int(patch_a))
    core_b = (int(major_b), int(minor_b), int(patch_b))
    if core_a < core_b:
        return -1
    if core_a > core_b:
        return 1

    # 2. Core versions are equal. Handle pre-release vs release precedence.
    if pre_a is None and pre_b is None:
        return 0
    if pre_a is not None and pre_b is None:
        return -1  # a has pre-release, b does not -> a < b
    if pre_a is None and pre_b is not None:
        return 1   # b has pre-release, a does not -> a > b

    # 3. Both have pre-release identifiers. Compare dot-by-dot.
    ids_a = pre_a.split('.')
    ids_b = pre_b.split('.')

    for id_a, id_b in zip(ids_a, ids_b):
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()

        if is_num_a and is_num_b:
            # Numeric identifiers compared numerically
            if int(id_a) < int(id_b):
                return -1
            if int(id_a) > int(id_b):
                return 1
        elif is_num_a:
            # Numeric always ranks lower than alphanumeric
            return -1
        elif is_num_b:
            return 1
        else:
            # Alphanumeric identifiers compared lexically (ASCII)
            if id_a < id_b:
                return -1
            if id_a > id_b:
                return 1

    # 4. All compared identifiers are equal. Larger set of fields has higher precedence.
    if len(ids_a) < len(ids_b):
        return -1
    if len(ids_a) > len(ids_b):
        return 1
        
    return 0


class TestCompareVersions(unittest.TestCase):
    def test_pre_release_vs_release(self):
        """A version with a pre-release tag has LOWER precedence than the same version without one."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-beta"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        """Numeric identifiers always rank lower than alphanumeric ones."""
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-1", "1.0.0-alpha"), -1)

    def test_differing_field_counts(self):
        """When all preceding identifiers are equal, a larger set of fields ranks higher."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.1.1"), -1)

    def test_build_metadata_ignored(self):
        """Build metadata (after '+') is completely ignored in precedence comparison."""
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+build", "1.0.0-beta+build"), -1)

    def test_alphanumeric_lexical_order(self):
        """Alphanumeric identifiers are compared lexically in ASCII order."""
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-beta"), -1)
        self.assertEqual(compare_versions("1.0.0-1.0", "1.0.0-1.1"), -1)

    def test_core_version_priority(self):
        """Major, minor, and patch take precedence over pre-release and build metadata."""
        self.assertEqual(compare_versions("2.0.0", "1.999.999"), 1)
        self.assertEqual(compare_versions("0.1.0", "0.0.9"), 1)


if __name__ == '__main__':
    unittest.main()
```

### Key Implementation Details:
1. **Regex Parsing**: Uses `fullmatch` to strictly validate SemVer format while capturing major/minor/patch, optional pre-release, and optional build metadata.
2. **Core Comparison**: Compares `(major, minor, patch)` as tuples. Python's tuple comparison is lexicographical and handles integer ordering perfectly.
3. **Pre-release Logic**: 
   - Explicitly handles the `None` cases for release vs pre-release.
   - Splits pre-release strings by `.` and iterates pairwise with `zip`.
   - Uses `.isdigit()` to distinguish numeric vs alphanumeric identifiers per spec.
   - Falls back to length comparison when all paired identifiers match.
4. **Build Metadata**: Captured by regex but explicitly ignored during comparison, satisfying the spec.
5. **Tests**: Exactly 6 test methods covering all requested scenarios, plus boundary cases to ensure robustness. All run via `unittest`.
