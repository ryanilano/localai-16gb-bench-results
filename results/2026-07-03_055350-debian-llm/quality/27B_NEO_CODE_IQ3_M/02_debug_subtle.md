# 27B_NEO_CODE_IQ3_M — 02_debug_subtle

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
The loop unconditionally increments `i` (`i += 1`) even after removing a duplicate with `nums.pop(i)`.

### Why It Fails
When `nums.pop(i)` is called, all elements after index `i` shift left by one position. The element that slides into index `i` now needs to be compared against `nums[i-1]` to catch longer runs of duplicates (e.g., three or more adjacent identical values). Because `i` is incremented regardless of whether a pop occurred, the newly shifted element is completely skipped, leaving extra duplicates in the list.

### Minimal Failing Input
```python
nums = [1, 1, 1]
```
- **Expected:** `[1]`, returns `1`
- **Actual:** `[1, 1]`, returns `2`  
*(Trace: `i=1` pops the second `1`, the third `1` shifts to index `1`. `i` increments to `2`, the loop condition `2 < 2` fails, and the function exits without comparing the remaining duplicate.)*

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
**No, the fix does not change the asymptotic complexity.** The original algorithm already has a worst-case time complexity of **O(n²)** due to `list.pop(i)` shifting subsequent elements in memory. The corrected version still performs at most `n` iterations and uses the exact same `pop(i)` operation, so it remains O(n²) worst-case (or O(n) if counting only comparisons). Auxiliary space complexity remains O(1) as it modifies the list in place.
