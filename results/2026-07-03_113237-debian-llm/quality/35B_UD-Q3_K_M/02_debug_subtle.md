# 35B_UD-Q3_K_M — 02_debug_subtle

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

### 🔍 Exact Bug
The bug is the unconditional `i += 1` at the end of the loop. When `nums.pop(i)` is executed, the list shrinks and all elements after index `i` shift left by one position. The element that was originally at `i+1` now occupies index `i`, but the code immediately increments `i`, skipping the newly shifted element.

### 📉 Why It Fails
`list.pop(i)` in Python shifts all subsequent elements left to fill the gap. By incrementing `i` after a pop, the algorithm moves past the element that just slid into position `i`. This means consecutive duplicates beyond the second one are never compared to their new predecessor, leaving unwanted duplicates in the list.

### 🧪 Minimal Input
```python
[1, 1, 1]
```
- **Expected result:** `1` (list becomes `[1]`)
- **Actual result:** `2` (list becomes `[1, 1]` and loop exits prematurely)

### ✅ Corrected Version
```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
        else:
            i += 1  # Only advance when no element is removed
    return len(nums)
```
This preserves the original algorithmic structure (single pass with index tracking and in-place popping) while fixing the index drift.

### ⏱️ Complexity Impact
**No change.** The fix does not alter the asymptotic complexity:
- **Time:** `O(N²)` worst-case. Each `pop(i)` shifts remaining elements, and in the worst case (e.g., all duplicates), we perform `O(N)` pops, each taking `O(N)` time.
- **Space:** `O(1)` auxiliary space (modifies list in-place).
The corrected version performs the exact same number of comparisons and `pop` operations as the original, so the complexity remains identical.
