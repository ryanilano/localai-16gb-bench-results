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
The bug is the **unconditional increment of the index `i`** (`i += 1`) even after an element has been removed from the list.

### Why It Fails
When `nums.pop(i)` is executed, the list shrinks, and all elements to the right of index `i` shift one position to the left. Specifically, the element that was at `i + 1` moves into index `i`.

However, the code immediately executes `i += 1`. This causes the loop to move to the next index, effectively **skipping the check** for the element that just shifted into position `i`. If that element was also a duplicate of `nums[i-1]`, it will be missed.

### Minimal Input
**Input:** `[1, 1, 1]`

**Trace of Failure:**
1.  `i = 0`: Check skipped (`i > 0` is false). `i` increments to 1.
2.  `i = 1`: `nums[1]` (1) equals `nums[0]` (1). `pop(1)` removes the middle 1. List becomes `[1, 1]`.
3.  `i` increments to 2.
4.  Loop condition `while 2 < len([1, 1])` (which is 2) evaluates to **False**.
5.  Loop terminates. The list remains `[1, 1]`, but it should be `[1]`.

### Corrected Version
To fix this, `i` should only increment if an element was **not** removed. If an element is removed, `i` should stay the same so the next iteration checks the new element at that index.

```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
            # Do NOT increment i here; the next element shifted into this index
        else:
            i += 1
    return len(nums)
```

### Complexity
**Does the fix change the O() complexity?**
**No.** The time complexity remains **O(N²)** in the worst case.

**Reasoning:** The dominant factor is the `nums.pop(i)` operation. In Python lists (which are dynamic arrays), removing an element at index `i` requires shifting all subsequent elements to the left, which takes O(K) time where K is the number of elements after `i`. Since this happens inside a loop that runs up to N times, the total complexity remains O(N²). The fix only alters the control flow (when `i` increments), not the cost of the operations performed.
