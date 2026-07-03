# 27B_HauhauCS_Balanced — 02_debug_subtle

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
The bug is that the index `i` is incremented unconditionally (`i += 1`) at the end of every loop iteration, even when an element has just been removed from the list.

### Why It Fails
When `nums.pop(i)` is executed, the element at index `i` is removed, and all subsequent elements shift one position to the left (the element that was at `i + 1` moves into index `i`).

Because the code immediately increments `i` to `i + 1`, the loop skips over the element that just shifted into position `i`. This new element is never compared against its predecessor (`nums[i - 1]`), causing adjacent duplicates to remain in the list.

### Minimal Input
The minimal input that triggers this bug is a list with **three** identical adjacent elements:
```python
[1, 1, 1]
```
**Trace:**
1. `i=0`: No match. `i` becomes 1.
2. `i=1`: `nums[1]` (1) matches `nums[0]` (1). `pop(1)` removes the middle 1. List is now `[1, 1]`.
3. `i` increments to 2.
4. Loop check `2 < len([1, 1])` (2 < 2) fails. Loop ends.
5. Result: `[1, 1]` (Incorrect; should be `[1]`).

### Corrected Code
To fix this, `i` should only be incremented if an element was *not* removed. If an element is removed, `i` must stay the same to check the newly shifted element in the next iteration.

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

### Complexity Analysis
**Does the fix change the O() complexity?**
**No.** The time complexity remains **$O(N^2)$**.

*   **Reasoning:** The `list.pop(i)` operation in Python takes $O(K)$ time (where $K$ is the number of elements after index `i`) because it must shift the remaining elements to fill the gap. In the worst-case scenario (e.g., a list of all identical elements `[1, 1, ..., 1]`), the loop runs $N$ times and performs a shift operation of size roughly $N$ each time, resulting in quadratic time complexity $O(N^2)$. The logic fix does not alter the underlying cost of the `pop` operation.
