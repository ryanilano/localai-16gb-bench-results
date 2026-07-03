# 27B_Heretic_NEO_CODE_IQ4_XS — 02_debug_subtle

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

**Exact bug:**
After removing a duplicate with `nums.pop(i)`, the code unconditionally executes `i += 1`. This causes it to skip the element that just shifted into index `i`, so adjacent duplicates can be missed.

**Why it fails:**
`pop(i)` shifts all elements to the right one position left. The new element at index `i` must be compared to `nums[i-1]`, but because `i` is incremented immediately, the loop moves past it without checking. This leads to leftover adjacent duplicates (and in some patterns, can also cause index-out-of-range or incorrect early termination).

**Minimal failing input:**
- Input: `[1, 1, 1]`
- Expected: `[1]`, returns `1`
- Actual: `[1, 1]`, returns `2` (the third `1` is never checked against the second)

**Corrected version (same algorithm, fixed indexing):**
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

**Complexity impact:**
No change. Both versions:
- Time: O(n²) worst case (due to `pop(i)` shifting elements)
- Space: O(1) extra (in-place)
The fix only adjusts when `i` is incremented; it does not alter the underlying operation count or asymptotic complexity.
