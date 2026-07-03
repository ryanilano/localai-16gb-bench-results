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
After `nums.pop(i)` removes an element, all subsequent elements shift one index to the left. The element that was originally at `i+1` now sits at index `i`. However, the code unconditionally executes `i += 1` at the end of every loop iteration. This causes the loop to **skip checking the newly shifted element** against its left neighbor.

### 📉 Why It Fails
When consecutive duplicates exist, the first duplicate is correctly removed, but the index increments past the second duplicate before it can be compared to the remaining first duplicate. As a result, the second duplicate is left in place, breaking the "remove all adjacent duplicates" guarantee.

### 🧪 Minimal Input
`[2, 2, 2]`
- **Expected:** `1` (list becomes `[2]`)
- **Actual:** `2` (list becomes `[2, 2]`)
**Trace:** 
- `i=0`: skip → `i=1`
- `i=1`: `nums[1]==nums[0]` → `pop(1)` → list `[2, 2]`, length becomes 2. `i` increments to `2`.
- `i=2`: `i < len(nums)` (2 < 2) is `False` → loop exits. The second `2` is never compared to the first.

### ✅ Corrected Version
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
**Fix:** Only increment `i` when no element is popped. This ensures the newly shifted element at index `i` is compared against `nums[i-1]` in the next iteration.

### ⏱️ Complexity Impact
**No change.** The fix does not alter the asymptotic complexity:
- **Time:** Remains **O(N²)** in the worst case because `list.pop(i)` requires shifting all subsequent elements, and up to `N` pops may occur.
- **Space:** Remains **O(1)** since the operation modifies the list in-place without allocating auxiliary structures.
