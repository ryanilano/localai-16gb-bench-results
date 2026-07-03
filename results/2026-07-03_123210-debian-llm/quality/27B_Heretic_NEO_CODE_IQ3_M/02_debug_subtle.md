# 27B_Heretic_NEO_CODE_IQ3_M — 02_debug_subtle

## Prompt

The function below is meant to remove all adjacent duplicate values from a list in place and return the count of remaining elements. It has a subtle bug that causes it to skip elements or crash on some inputs. Identify the exact bug, explain why it fails, give a minimal input that triggers the wrong result, and provide a corrected version.

```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
        i += 1
    return len(nums)
```

Do not rewrite it with a different algorithm — fix the existing one, and state whether your fix changes the O() complexity.

## Response

**Bug:**
The unconditional `i += 1` after removing an element.

**Why it fails:**
When `nums.pop(i)` runs, all elements to the right shift left by one. The element that moves into index `i` is now adjacent to `i-1` and must be checked. Because `i` is incremented immediately, the loop skips this newly adjacent element. As a result, runs of three or more identical values are not fully removed. (In stricter environments or with minor condition changes, this same off-by-one pattern can easily lead to out-of-bounds access, which is likely what “crash” refers to.)

**Minimal failing input:**
`[1, 1, 1]`
- Expected: `[1]`, length `1`
- Actual (current code): `[1, 1]`, length `2`

**Corrected version (same algorithm style):**
```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
        else:
            i += 1
    return len(nums)
```

**Complexity:**
No change. The original and fixed versions both use `pop(i)` on a Python list, which shifts elements and costs O(k) time. In the worst case (all adjacent duplicates), we perform O(n) pops, each up to O(n), so the time complexity remains O(n²). The fix only alters control flow, not the underlying operations.
