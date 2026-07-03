# 35B_UD-Q4_K_XL — 02_debug_subtle

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

### Exact Bug
After `nums.pop(i)` removes an element, the list shrinks and all subsequent elements shift left by one index. The code then immediately increments `i` (`i += 1`), causing the loop to **skip the element that just moved into index `i`**. The loop never re-evaluates the current index against its new left neighbor.

### Why It Fails
When consecutive duplicates appear (e.g., three or more identical values in a row), the first pair is correctly removed. However, because `i` advances past the newly shifted duplicate, the second pair is never compared against its left neighbor. The loop simply moves on, leaving unwanted duplicates in the list.

### Minimal Input That Triggers Wrong Result
```python
[1, 1, 1]
```
- **Expected result:** `1` (list becomes `[1]`)
- **Actual result:** `2` (loop removes one `1`, skips the second comparison, and exits)

### Corrected Version
```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
            i -= 1  # Cancel out the upcoming i += 1 to recheck the current index
        i += 1
    return len(nums)
```

### Complexity Analysis
**No, the fix does not change the asymptotic complexity.**  
The original algorithm runs in **O(n²)** worst-case time because `list.pop(i)` requires shifting O(n) elements, and this operation can occur up to O(n) times. The fix only adds a constant-time index adjustment, preserving the same O(n²) worst-case complexity (though it remains O(n) best-case when no duplicates exist).
