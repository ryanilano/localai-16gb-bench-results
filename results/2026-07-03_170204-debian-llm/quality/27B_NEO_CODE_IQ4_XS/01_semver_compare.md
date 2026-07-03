# 27B_NEO_CODE_IQ4_XS — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

Here's a complete implementation following the SemVer 2.0.0 specification, along with 6 targeted unit tests:

```python
import unittest

def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """
    def parse(v: str):
        # Build metadata is ignored for precedence
        v = v.split('+')[0]
        # Split core version and pre-release
        if '-' in v:
            core, pre = v.split('-', 1)
        else:
            core, pre = v, None
        major, minor, patch = map(int, core.split('.'))
        return major, minor, patch, pre

    ma, mia, pa, pra = parse(a)
    mb, mib, pb, prb = parse(b)

    # 1. Compare major, minor, patch numerically
    if ma != mb:
        return 1 if ma > mb else -1
    if mia != mib:
        return 1 if mia > mib else -1
    if pa != pb:
        return 1 if pa > pb else -1

    # 2. Handle pre-release precedence
    if pra is None and prb is None:
        return 0
    if pra is None:
        return 1  # a has no pre-release, so a > b
    if prb is None:
        return -1 # b has no pre-release, so a < b

    # 3. Compare pre-release identifiers dot-by-dot
    ids_a = pra.split('.')
    ids_b = prb.split('.')

    for id_a, id_b in zip(ids_a, ids_b):
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()

        if is_num_a and is_num_b:
            val_a, val_b = int(id_a), int(id_b)
            if val_a != val_b:
                return 1 if val_a > val_b else -1
        elif is_num_a:
            return -1  # numeric < alphanumeric
        elif is_num_b:
            return 1   # alphanumeric > numeric
        else:
            if id_a != id_b:
                return 1 if id_a > id_b else -1  # ASCII lexical order

    # 4. Larger set of fields ranks higher when all preceding ones are equal
    if len(ids_a) != len(ids_b):
        return 1 if len(ids_a) > len(ids_b) else -1
    return 0


class TestCompareVersions(unittest.TestCase):
    def test_prerelease_vs_release(self):
        # A version with a pre-release tag has lower precedence than the same version without one
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "1.0.0-beta"), 1)

    def test_numeric_vs_alphanumeric_identifiers(self):
        # Numeric identifiers always rank lower than alphanumeric ones
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.beta"), -1)
        self.assertEqual(compare_versions("1.0.0-1", "1.0.0-alpha"), -1)

    def test_differing_field_counts(self):
        # A larger set of fields ranks higher when all preceding ones are equal
        self.assertEqual(compare_versions("1.0.0-alpha", "1.0.0-alpha.1"), -1)
        self.assertEqual(compare_versions("1.0.0-alpha.1", "1.0.0-alpha.1.2"), -1)

    def test_build_metadata_ignored(self):
        # Build metadata is completely ignored in precedence
        self.assertEqual(compare_versions("1.0.0+build1", "1.0.0+build2"), 0)
        self.assertEqual(compare_versions("1.0.0-alpha+exp.sha.5114f85", "1.0.0-alpha+001"), 0)

    def test_basic_major_minor_patch(self):
        # Core version comparison follows standard numeric precedence
