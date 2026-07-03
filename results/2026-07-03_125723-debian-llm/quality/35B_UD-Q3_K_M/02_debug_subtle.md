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

### 🔍 Bug Identification
The loop unconditionally increments `i` (`i += 1`) after potentially removing an element with `nums.pop(i)`. When `pop(i)` is called, the list shrinks by one, and all elements after index `i` shift left by one position. The element that was at `i+1` now occupies index `i`, but incrementing `i` causes the loop to skip checking this newly shifted element against its predecessor.

### 📉 Why It Fails
This skipping behavior leaves consecutive duplicates unremoved. After removing the second duplicate, the third shifts into the current index but is never compared to the first, causing the algorithm to terminate early with an incorrect count.

### 🧪 Minimal Input That Triggers the Bug
```python
[1, 1, 1]
```
**Trace of original code:**
- `i=0`: no pop → `i` becomes 1
- `i=1`: `nums[1] == nums[0]` → `pop(1)` removes second `1` → list becomes `[1, 1]`, `i` becomes 2
- `i=2`: `2 < len([1, 1])` is `False` → loop ends
- Returns `2` instead of the correct `1`.

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
**Fix:** Only advance `i` when no removal occurs. This ensures the element that shifted into index `i` is properly compared against `nums[i-1]` in the next iteration.

### ⏱️ Complexity Impact
**No, the fix does not change the asymptotic complexity.**
- **Time Complexity:** Remains `O(N²)` in the worst case (e.g., all elements are identical). This is inherent to `list.pop(i)`, which requires shifting `O(N)` elements in Python's underlying array implementation.
- **Space Complexity:** Remains `O(1)` auxiliary, as the operation modifies the list in-place.
The structural approach (single-pass linear scan with in-place removal) is preserved; only the index control logic is corrected.
