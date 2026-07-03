# 27B_HauhauCS_Balanced_Q3_K_P — 02_debug_subtle

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

### 1. The Exact Bug
The bug is an **index skip** caused by incrementing the loop counter (`i`) immediately after removing an element from the list.

When `nums.pop(i)` is executed, all elements to the right of index `i` shift one position to the left. The element that was at `i + 1` moves into position `i`. However, the code immediately executes `i += 1`, moving the pointer to the next index. This causes the algorithm to skip checking the newly shifted element at position `i` against its predecessor.

### 2. Why It Fails
This logic works for isolated duplicates (e.g., `[1, 1, 2]`) but fails for sequences of three or more identical adjacent values. The algorithm removes the middle duplicate, shifts the third duplicate into the current slot, but then moves the pointer past it, leaving the third duplicate in the list unchecked.

### 3. Minimal Failing Input
The minimal input that triggers the wrong result is a list with three consecutive identical values, such as:
**`[1, 1, 1]`**

**Execution Trace for `[1, 1, 1]`:**
1.  `i=0`: `i` increments to 1.
2.  `i=1`: `nums[1]` (1) equals `nums[0]` (1). `pop(1)` is called. The list becomes `[1, 1]`.
3.  `i` increments to 2.
4.  `i=2`: The loop condition `i < len(nums)` (2 < 2) is now false. The loop terminates.
5.  **Result:** The function returns `[1, 1]` (length 2), but the expected result is `[1]` (length 1).

### 4. Corrected Version
To fix this, we must ensure that if an element is removed, the index `i` remains the same so the newly shifted element is compared in the next iteration.

```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
            # Fix: Do not increment i if a deletion occurred
        else:
            i += 1
    return len(nums)
```

### 5. Complexity Analysis
**Does the fix change the complexity?**
**No.** The time complexity remains **$O(N^2)$** in the worst case.

*   **Reasoning:** In Python, `list.pop(i)` is an $O(K)$ operation where $K$ is the number of elements to the right of the removal index (due to the shifting of elements). In a worst-case scenario where the list contains $N$ identical elements (e.g., `[1, 1, 1, ... 1]`), the algorithm performs $N$ removals. The cost of these operations sums to $N + (N-1) + (N-2) + ... + 1$, which results in quadratic time complexity.
*   **Note:** While a two-pointer approach (overwriting elements) would achieve $O(N)$, the constraint to fix the *existing* algorithm (which relies on `pop`) means the complexity class remains unchanged.
