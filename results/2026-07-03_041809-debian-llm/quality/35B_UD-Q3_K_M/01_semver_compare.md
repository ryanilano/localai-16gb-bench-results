# 35B_UD-Q3_K_M — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

The user wants a function `compare_versions(a: str, b: str) -> int` that implements Semantic Versioning 2.0.0 comparison logic.

Key rules to implement:
1.  Compare Major, Minor, Patch numerically.
2.  Pre-release vs Release: Pre-release has lower precedence than release (if major/minor/patch are equal).
3.  Pre-release comparison:
    *   Split by `.`.
    *   Compare identifiers one by one.
    *   Numeric identifiers are compared as integers.
    *   Alphanumeric identifiers are compared as strings (ASCII order).
    *   Numeric identifiers always have lower precedence than alphanumeric identifiers (if one is numeric and the other is alphanumeric).
    *   If identifiers are equal, move to the next.
    *   If all preceding identifiers are equal, the one with more fields has higher precedence.
    *   If one runs out of fields first, the shorter one has lower precedence.
4.  Build metadata (after `+`) is ignored.

Let's break down the parsing logic.
A SemVer string looks like `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`.
Regex or string splitting can be used.
Format: `^(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?(?:\+([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?$`

Steps:
1.  Parse version strings into components: Major, Minor, Patch, Pre-release list, Build metadata (ignored).
2.  Compare Major, Minor, Patch. If not equal, return result.
3.  If equal, check Pre-release status.
    *   If `a` has pre-release and `b` does not: `a < b` (-1).
    *   If `b` has pre-release and `a` does not: `a > b` (1).
    *   If both have pre-release, compare the lists of identifiers.
    *   If neither has pre-release: `a == b` (0).

Pre-release comparison logic details:
Given two lists of identifiers `pre_a` and `pre_b`.
Iterate index `i` from 0 to `min(len(pre_a), len(pre_b)) - 1`.
Let `id_a = pre_a[i]`, `id_b = pre_b[i]`.
Determine type of `id_a` (numeric or alphanumeric).
Determine type of `id_b` (numeric or alphanumeric).

Comparison cases:
1.  Both numeric: compare integers.
2.  Both alphanumeric: compare strings.
3.  One numeric, one alphanumeric: numeric is lower.

If `id_a` != `id_b` in value, return result.
If loop finishes without difference:
Check lengths.
If `len(pre_a) > len(pre_b)`: `a > b` (1).
If `len(pre_a) < len(pre_b)`: `a < b` (-1).
Else: `a == b` (0).

Let's refine the parsing.
Split string by `-` first to separate main version and pre-release?
Actually, `+` comes after `-`? No, SemVer is `MAJOR.MINOR.PATCH` followed optionally by `-prerelease` then optionally `+build`.
Wait, the regex structure is `MAJOR.MINOR.PATCH` then optional `-` then prerelease then optional `+` then build.
So splitting by `-` might be tricky if prerelease contains hyphens? No, prerelease identifiers can contain hyphens.
Better to use regex or careful string manipulation.
Regex: `^(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?(?:\+([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?$`

Let's write a parser helper.

Unit tests required:
1.  Pre-release vs Release: `1.0.0-alpha` vs `1.0.0`. Expected -1.
2.  Numeric vs Alphanumeric identifiers: `1.0.0-alpha.1` vs `1.0.0-alpha.beta`.
    *   `alpha` == `alpha`.
    *   `1` (numeric) vs `beta` (alphanumeric). Numeric < Alphanumeric.
    *   Expected -1.
3.  Differing field counts: `1.0.0-alpha` vs `1.0.0-alpha.1`.
    *   `alpha` == `alpha`.
    *   `a` has no more fields, `b` has `1`.
    *   Shorter < Longer.
    *   Expected -1.
4.  Build metadata ignored: `1.0.0-alpha+001` vs `1.0.0-alpha+002`.
    *   Major/minor/patch equal.
    *   Pre-release equal.
    *   Build ignored.
    *   Expected 0.
5.  Maybe standard numeric comparison: `1.0.0` vs `2.0.0`.
6.  Maybe pre-release ordering: `1.0.0-alpha` vs `1.0.0-beta`? Or `1.0.0-1` vs `1.0.0-2`.

Let's draft the code.

```python
import re

def compare_versions(a: str, b: str) -> int:
    # Regex for SemVer
    # Groups: 1:Major, 2:Minor, 3:Patch, 4:Pre-release string, 5:Build string
    regex = r'^(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?(?:\+([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?$'
    
    m_a = re.match(regex, a)
    m_b = re.match(regex, b)
    
    if not m_a or not m_b:
        raise ValueError("Invalid Semantic Version string")

    # Parse components
    def parse(ver_match):
        major = int(ver_match.group(1))
        minor = int(ver_match.group(2))
        patch = int(ver_match.group(3))
        pre_str = ver_match.group(4)
        # build = ver_match.group(5) # Ignored
        
        pre_list = []
        if pre_str:
            pre_list = pre_str.split('.')
            
        return major, minor, patch, pre_list

    maj_a, min_a, pat_a, pre_a = parse(m_a)
    maj_b, min_b, pat_b, pre_b = parse(m_b)

    # Compare Major, Minor, Patch
    if maj_a != maj_b:
        return -1 if maj_a < maj_b else 1
    if min_a != min_b:
        return -1 if min_a < min_b else 1
    if pat_a != pat_b:
        return -1 if pat_a < pat_b else 1

    # If major/minor/patch are equal, compare pre-release
    # Rule: A version WITH a pre-release tag has LOWER precedence than the same version without one
    if pre_a and not pre_b:
        return -1
    if not pre_a and pre_b:
        return 1
    
    # If neither has pre-release, they are equal
    if not pre_a and not pre_b:
        return 0

    # Both have pre-release, compare identifiers
    # Rule: Compare dot-by-dot
    # Rule: Numeric identifiers compared numerically
    # Rule: Alphanumeric compared lexically
    # Rule: Numeric identifiers always rank lower than alphanumeric ones
    # Rule: Larger set of fields ranks higher when all preceding ones are equal
    
    len_a = len(pre_a)
    len_b = len(pre_b)
    min_len = min(len_a, len_b)
    
    for i in range(min_len):
        id_a = pre_a[i]
        id_b = pre_b[i]
        
        is_num_a = id_a.isdigit()
        is_num_b = id_b.isdigit()
        
        # Check if numeric
        # Note: SemVer spec says identifiers MUST comprise only ASCII alphanumerics and hyphen [0-9A-Za-z-].
        # Also "Numeric identifiers MUST NOT include leading zeroes."
        # However, for comparison, checking if it's purely digits is usually sufficient for type check.
        # The spec says "Numeric identifiers are compared as integers".
        # "Alphanumeric identifiers are compared lexically in ASCII order".
        # "Numeric identifiers always rank lower than alphanumeric identifiers".
        
        if is_num_a and is_num_b:
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a != val_b:
                return -1 if val_a < val_b else 1
        elif is_num_a and not is_num_b:
            # Numeric < Alphanumeric
            return -1
        elif
