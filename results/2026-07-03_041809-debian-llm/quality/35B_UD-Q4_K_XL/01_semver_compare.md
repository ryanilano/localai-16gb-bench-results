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

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process:

1.  **Understand the Goal**: The user wants a Python function `compare_versions(a: str, b: str) -> int` that compares two Semantic Versioning 2.0.0 strings according to the exact precedence rules specified. It must return -1, 0, or 1. I also need to provide 6 unit tests covering specific cases.

2.  **Analyze SemVer 2.0.0 Rules**:
   - Format: `MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]`
   - Compare MAJOR, MINOR, PATCH numerically.
   - Pre-release has lower precedence than the same version without it.
   - Pre-release comparison:
     - Split by `.`
     - Compare identifiers one by one.
     - Numeric identifiers compared numerically.
     - Alphanumeric identifiers compared lexically (ASCII order).
     - Numeric identifiers always have lower precedence than alphanumeric.
     - A version with fewer fields has lower precedence if all preceding fields are equal.
   - Build metadata (after `+`) is ignored.

3.  **Design the Comparison Logic**:
   - Strip build metadata first: split on `+`, take the first part.
   - Split the version string into core version and pre-release part using `-`. Note: The core version is `MAJOR.MINOR.PATCH`. Pre-release is everything after the first `-`.
   - Parse core version: split by `.`, convert to integers.
   - Parse pre-release: if present, split by `.`. Each identifier needs to be checked if it's purely numeric.
   - Comparison steps:
     1. Compare core versions (major, minor, patch). If any differ, return -1 or 1.
     2. If core versions are equal:
        - If both have no pre-release: return 0.
        - If one has pre-release and the other doesn't: the one without has higher precedence -> return 1 or -1.
        - If both have pre-release: compare identifier by identifier.
          - For each pair of identifiers:
            - Check if both are numeric: compare numerically.
            - Check if one is numeric and the other is alphanumeric: numeric has lower precedence.
            - If both are alphanumeric: compare lexically (string comparison).
            - If one runs out of identifiers: the shorter one has lower precedence.
          - If all identifiers are equal: return 0.

4.  **Handle Edge Cases & Details**:
   - What about leading zeros? SemVer says numeric identifiers MUST NOT include leading zeroes. I'll assume valid input per the prompt, but I should handle numeric parsing correctly.
   - Alphanumeric comparison: ASCII order. Python's default string comparison is lexicographical based on Unicode code points, which matches ASCII for standard characters. I'll stick to standard string comparison.
   - Numeric vs Alphanumeric: "Numeric identifiers always have lower precedence than alphanumeric identifiers." So if `a_id` is numeric and `b_id` is alphanumeric, `a` < `b`.
   - Pre-release splitting: `version.rsplit('-', 1)` is safer because pre-release identifiers themselves can contain hyphens? Wait, SemVer spec says pre-release identifiers are separated by dots, and identifiers are alphanumeric and hyphens. Actually, the spec says: "Pre-release versions have a lower precedence than the associated normal version." and "Pre-release identifiers are separated by a dot (`.`)." Hyphens are allowed within identifiers. So splitting on `-` only once is correct.
   - Let's verify: `1.0.0-alpha-beta` -> core `1.0.0`, pre-release `alpha-beta`. Splitting on `-` with `maxsplit=1` works.
   - Build metadata: `1.0.0+build` -> split on `+`, take `[0]`.
   - So: `clean = v.split('+')[0]`
   - Then: `core_str, _, pre_str = clean.partition('-')`
   - Core: `core_str.split('.')` -> map to int.
   - Pre: `pre_str.split('.')` if `pre_str` else `[]`.
   - For each pre identifier: check `isdigit()`. Note: `isdigit()` might not catch all numeric strings (e.g., negative numbers aren't allowed in SemVer pre-release, so `isdigit()` or `int()` check is fine). I'll use `ident.isdigit()` or try/except. Actually, `ident.isnumeric()` or `ident.isdigit()` is fine. I'll stick to `ident.isdigit()`.

5.  **Draft the Code**:
   ```python
   def compare_versions(a: str, b: str) -> int:
       def parse_version(v: str):
           # Ignore build metadata
           core_part = v.split('+')[0]
           # Split core version and pre-release
           if '-' in core_part:
               core_str, pre_str = core_part.split('-', 1)
           else:
               core_str, pre_str = core_part, ""

           # Parse core version
           major, minor, patch = map(int, core_str.split('.'))

           # Parse pre-release identifiers
           pre_ids = pre_str.split('.') if pre_str else []
           # Convert to tuple of (is_numeric, value)
           # For numeric: (True, int_val)
           # For alphanumeric: (False, str_val)
           parsed_pre = []
           for ident in pre_ids:
               if ident.isdigit():
                   parsed_pre.append((True, int(ident)))
               else:
                   parsed_pre.append((False, ident))
           return (major, minor, patch), parsed_pre

       core_a, pre_a = parse_version(a)
       core_b, pre_b = parse_version(b)

       # Compare core versions
       if core_a < core_b:
           return -1
       elif core_a > core_b:
           return 1

       # Core versions are equal, compare pre-release
       # If both are empty, they are equal
       if not pre_a and not pre_b:
           return 0
       # If one has pre-release and the other doesn't, the one without is higher
       if not pre_a:
           return 1
       if not pre_b:
           return -1

       # Both have pre-release, compare identifier by identifier
       for id_a, id_b in zip(pre_a, pre_b):
           is_num_a, val_a = id_a
           is_num_b, val_b = id_b

           if is_num_a and is_num_b:
               if val_a < val_b:
                   return -1
               elif val_a > val_b:
                   return 1
           elif is_num_a and not is_num_b:
               # Numeric has lower precedence than alphanumeric
               return -1
           elif not is_num_a and is_num_b:
               return 1
           else:
               # Both alphanumeric, compare lexically
               if val_a < val_b:
                   return -1
               elif val_a > val_b:
                   return 1

       # All compared identifiers are equal
       # The one with more identifiers has higher precedence
       if len(pre_a) < len(pre_b):
           return -1
       elif len(pre_a) > len(pre_b):
           return 1
       return 0
   ```

6.  **Verify against SemVer Rules**:
   - `1.0.0-alpha < 1.0.0` -> `pre_a` has items, `pre_b` empty -> returns -1. Correct.
   - `1.0.0-alpha.1 < 1.0.0-alpha.beta` -> `alpha` vs `alpha` (equal), `1` (numeric) vs `beta` (alphanumeric) -> numeric < alphanumeric -> returns -1. Correct.
   - `1.0.0-alpha < 1.0.0-alpha.1` -> `alpha` vs `alpha` (equal), `pre_a` ends, `pre_b` continues -> shorter has lower precedence -> returns -1. Correct.
   - Build metadata ignored: `split('+')[0]` handles it. `1.0.0+build1` vs `1.0.0+build2` -> core equal, pre empty -> returns 0. Correct.
   - ASCII lexical order: Python string comparison uses Unicode code points, which for ASCII matches lexical order. Correct.
   - Numeric vs Alphanumeric precedence: Spec says "Numeric identifiers always have lower precedence than alphanumeric identifiers." Handled.
   - One detail: `isdigit()` might return True for some unicode digits, but SemVer is ASCII. I'll use `ident.isascii() and ident.isdigit()` or just `int(ident)` in a try/except, or `ident.isdigit()` is fine for typical inputs. To be strictly compliant, I'll check if it matches `^[0-9]+$`. Actually, `ident.isdigit()` is standard enough. I'll stick with it, or use `ident.isdecimal()`. I'll just use `ident.isdigit()`.

   Wait, let's double check the exact wording: "numeric identifiers compared numerically, alphanumeric compared lexically in ASCII order, numeric identifiers always rank lower than alphanumeric ones".
   My logic handles this.

   One edge case: `1.0.0-01` vs `1.0.0-1`. SemVer says numeric
