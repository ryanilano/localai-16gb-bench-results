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

### 🔍 The Bug
The unconditional `i += 1` after `nums.pop(i)`.

### 📖 Why It Fails
`list.pop(i)` removes the element at index `i` and **shifts all subsequent elements one position to the left**. The element that was originally at `i+1` is now at index `i`. By immediately executing `i += 1`, the loop skips checking this newly shifted element against its new predecessor. This causes the algorithm to miss adjacent duplicates that form after a removal, leading to incorrect counts and leftover duplicates.

### 🧪 Minimal Failing Input
```python
[1, 1, 1]
```
**Trace with original code:**
- `i=0`: `i>0` is false → `i` becomes 1
- `i=1`: `nums[1] == nums[0]` → `pop(1)` removes middle `1`, list becomes `[1, 1]` → `i` becomes 2
- `i=2`: `len(nums)` is 2, so `2 < 2` is false → loop exits
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
*(Note: Keeping `i=0` and the `i>0` guard preserves your original structure. Starting at `i=1` would be a minor optimization but isn't required for correctness.)*

### ⏱️ Complexity Impact
**The fix does not change the O() complexity.** 
- **Time:** Still `O(N²)` in the worst case. `list.pop(i)` requires shifting all elements after `i`, which takes `O(k)` time. The control-flow fix only prevents skipping indices; it doesn't eliminate the costly shifts.
- **Space:** Still `O(1)` auxiliary space, as modifications are done in-place.

The correction strictly fixes the iteration logic while preserving the original algorithmic approach.
