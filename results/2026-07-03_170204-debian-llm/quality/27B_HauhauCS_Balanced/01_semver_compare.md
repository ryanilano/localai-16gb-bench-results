# 27B_HauhauCS_Balanced — 01_semver_compare

## Prompt

Implement a function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings. Return -1 if a < b, 0 if equal, 1 if a > b.

You MUST follow the SemVer precedence rules exactly:
- Compare major, minor, patch numerically.
- A version WITH a pre-release tag has LOWER precedence than the same version without one (1.0.0-alpha < 1.0.0).
- Compare pre-release identifiers dot-by-dot: numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones, and a larger set of fields ranks higher when all preceding ones are equal.
- Build metadata (after '+') is IGNORED in precedence.

Include 6 unit tests that specifically cover: pre-release vs release, numeric vs alphanumeric identifiers (e.g. 1.0.0-alpha.1 vs 1.0.0-alpha.beta), differing field counts (1.0.0-alpha vs 1.0.0-alpha.1), and that build metadata is ignored.

## Response

```python
def compare_versions(a: str, b: str) -> int:
    """
    Compares two Semantic Versioning 2.0.0 strings.
    Returns -1 if a < b, 0 if equal, 1 if a > b.
    """

    def parse_version(v: str):
        # 1. Remove build metadata (everything after '+')
        if '+' in v:
            v = v.split('+')[0]
        
        # 2. Split pre-release tag (everything after '-')
        if '-' in v:
            core_str, pre_str = v.split('-', 1)
            pre_ids = pre_str.split('.')
        else:
            core_str = v
            pre_ids = None  # Indicates a standard release version
        
        # 3. Parse Major, Minor, Patch
        # Assuming valid SemVer format, these are integers
        core_parts = list(map(int, core_str.split('.')))
        
        return core_parts, pre_ids

    # Parse both versions
    a_core, a_pre = parse_version(a)
    b_core, b_pre = parse_version(b)

    # 1. Compare Major, Minor, Patch numerically
    if a_core != b_core:
        if a_core < b_core:
            return -1
        return 1

    # 2. Compare Pre-release status
    # A version with a pre-release tag has lower precedence than the same version without one.
    if a_pre is None and b_pre is None:
        return 0  # Both are standard releases, and core parts are equal
    if a_pre is None and b_pre is not None:
        return 1  # a is release, b is pre-release -> a > b
    if a_pre is not None and b_pre is None:
        return -1 # a is pre-release, b is release -> a < b

    # 3. Compare Pre-release identifiers (both have pre-release tags)
    # Compare dot-by-dot
    for id_a, id_b in zip(a_pre, b_pre):
        a_is_num = id_a.isdigit()
        b_is_num = id_b.isdigit()

        if a_is_num and b_is_num:
            # Both numeric: compare as integers
            val_a = int(id_a)
            val_b = int(id_b)
            if val_a < val_b:
                return -1
            if val_a > val_b:
                return 1
        elif a_is_num and not b_is_num:
            # Numeric identifiers always have lower precedence than alphanumeric
            return -1
        elif not a_is_num and b_is_num:
            # Alphanumeric identifiers have higher precedence than numeric
            return 1
        else:
            # Both alphanumeric: compare lexically in ASCII order
            if id_a < id_b:
                return -1
            if id_a > id_b:
                return 1

    # 4. If all compared identifiers are equal, the one with more fields ranks higher
    if len(a_pre) < len(b_pre):
        return -1
    if len(a_pre) > len(b_pre):
        return 1

    return 0

# --- Unit Tests ---

if __name__ == "__main__":
    def test(name, v1, v2, expected):
        res = compare_versions(v1, v2)
        status = "PASS" if res == expected else "FAIL"
        print(f"[{status}] {name}: compare_versions('{v1}', '{v2}') -> {res} (Expected: {expected})")

    # 1. Pre-release vs Release (Pre-release is lower)
    test("Pre-release vs Release", "1.0.0-alpha", "1.0.0", -1)

    # 2. Numeric vs Alphanumeric identifiers (Numeric is lower)
    test("Numeric vs Alphanumeric ID", "1.0.0-alpha.1", "1.0.0-alpha.beta", -1)

    # 3. Differing field counts (More fields is higher)
    test("Field Count", "1.0.0-alpha", "1.0.0-alpha.1", -1)

    # 4. Build metadata is ignored
    test("Build Metadata Ignored", "1.0.0+build1", "1.0.0+build2", 0)

    # 5. Alphanumeric lexical comparison (ASCII order)
    test("Alphanumeric Lexical", "1.0.0-alpha", "1.0.0-beta", -1)

    # 6. Numeric identifier comparison (Integer value)
    test("Numeric Comparison", "1.0.0-1", "1.0.0-2", -1)
```
