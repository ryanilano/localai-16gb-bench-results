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
After removing a duplicate with `nums.pop(i)`, the code unconditionally executes `i += 1`. This causes the next element (which has just shifted left into index `i`) to be skipped entirely.

**Why it fails:**
When you pop at index `i`, all elements after it shift left by one. The element that needs to be compared next is now at the same index `i`, but because `i` is incremented, the loop moves on to `i+1`, never checking the newly shifted element against its predecessor. This leads to missed adjacent duplicates (and in some variants of this pattern, index errors).

**Minimal failing input:**
```python
[1, 1, 1]
```
- Expected: `[1]`, returns `1`
- Actual with bug: `[1, 1]`, returns `2`

**Corrected version (same structure):**
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
No change. The original algorithm is already `O(n²)` time in the worst case because `list.pop(i)` shifts up to `O(n)` elements. The fix only changes control flow, not the underlying operations, so it remains `O(n²)` time and `O(1)` extra space.
