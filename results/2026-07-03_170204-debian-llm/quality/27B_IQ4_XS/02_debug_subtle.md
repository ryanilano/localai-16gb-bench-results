# 27B_IQ4_XS — 02_debug_subtle

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

### The Bug
The code unconditionally increments `i` after `nums.pop(i)`. When an element is removed, all subsequent elements shift left by one index. By incrementing `i` immediately, the loop skips the element that just shifted into position `i`, failing to compare it with its new predecessor.

### Why It Fails
List mutation changes the indices of remaining elements. The algorithm assumes static indexing, but `pop(i)` invalidates that assumption. The skipped element might be a duplicate of `nums[i-1]`, causing adjacent duplicates to remain in the list.

### Minimal Failing Input
`[1, 1, 1]`

**Trace:**
1. `i=0`: `i>0` is false → `i` becomes 1
2. `i=1`: `nums[1] == nums[0]` → `pop(1)` → list becomes `[1, 1]`, `i` becomes 2
3. `i=2`: `2 < len(nums)` (2 < 2) is false → loop exits
4. Returns `2`, but the list is `[1, 1]` (expected `[1]` with count `1`)

### Corrected Version
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

### Complexity Impact
**No, the fix does not change the asymptotic complexity.** 
- **Time:** Both versions run `O(N)` iterations. However, `list.pop(i)` requires shifting all subsequent elements, making the worst-case time complexity `O(N²)` and best/average case `O(N)`. The fix only corrects index management without adding or removing any shifting operations.
- **Space:** Both remain `O(1)` auxiliary space since the list is modified in-place.
