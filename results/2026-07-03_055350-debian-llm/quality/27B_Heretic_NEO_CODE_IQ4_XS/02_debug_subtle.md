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
The unconditional `i += 1` after `nums.pop(i)` causes the loop to skip the element that shifts left into position `i`. When a duplicate is removed, the next element moves into the current index and must be compared with `i-1`, but `i` is incremented anyway, so that comparison is never made.

**Why it fails:**
- After `pop(i)`, all elements to the right shift left by one.
- The new element at index `i` is now adjacent to `i-1` and needs to be checked.
- Because `i` increments regardless, that newly shifted element is never compared, leaving some adjacent duplicates in the list.

**Minimal input that triggers wrong result:**
`[1, 1, 1]`
- Expected: `[1]`, length `1`
- Actual with bug: `[1, 1]`, length `2`

**Corrected version (same algorithm, fixed control flow):**
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
No change. The algorithm still uses `pop(i)`, which shifts elements and costs O(k) time, so worst-case time complexity remains O(n²). The fix only adjusts when `i` is incremented, not the underlying operations.
