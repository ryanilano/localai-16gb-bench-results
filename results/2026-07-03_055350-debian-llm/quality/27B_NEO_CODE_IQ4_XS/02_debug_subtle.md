# 27B_NEO_CODE_IQ4_XS — 02_debug_subtle

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
The unconditional `i += 1` at the end of the `while` loop.

### Why It Fails
When `nums.pop(i)` is called, the list shrinks and every element after index `i` shifts left by one position. The element that was originally at `i + 1` now occupies index `i`. However, the code immediately increments `i`, causing the loop to skip over this newly shifted element. As a result, any consecutive duplicates beyond the first pair are never compared to their predecessor, leading to missed removals.

### Minimal Input That Triggers the Bug
```python
[1, 1, 1]
```
- **Expected:** `[1]` (returns `1`)
- **Actual with bug:** `[1, 1]` (returns `2`)

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
The fix **does not change** the asymptotic complexity. 
- **Time:** Remains **O(N²)** in the worst case (e.g., all elements are identical) because `list.pop(i)` must shift all subsequent elements, which is an O(k) operation. 
- **Space:** Remains **O(1)** auxiliary space.
*(Note: To achieve O(N) time, you would need to switch to a two-pointer overwrite approach, but per your constraint, the original algorithm structure is preserved.)*
