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

### 🔍 Exact Bug
The unconditional `i += 1` after `nums.pop(i)` **skips the element that shifts into index `i`**.

### 📖 Why It Fails
When `nums.pop(i)` removes an element, all subsequent elements in the list shift left by one index. The element that was originally at `i+1` is now at `i`. Because the code increments `i` immediately after the deletion, the next iteration checks `nums[i+1]` (which was originally at `i+2`), completely bypassing the newly shifted element. This breaks the chain of adjacent duplicate detection.

### 🧪 Minimal Triggering Input
```python
dedupe_adjacent([1, 1, 1])
```
**Trace of original code:**
- `i=0`: `i>0` is false → `i` becomes 1
- `i=1`: `nums[1] == nums[0]` → `pop(1)` removes second `1`. List becomes `[1, 1]`. `i` becomes 2.
- `i=2`: `i < len(nums)` (2 < 2) is false → loop ends.
- **Returns:** `2` (Incorrect. Should be `1`.)

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
*(Alternatively, replace `else: i += 1` with `continue` inside the `if` block. Both are minimal fixes that preserve the original algorithm.)*

### ⏱️ Complexity Impact
**No change.** The fix does not alter the algorithmic approach. 
- **Time Complexity:** Remains `O(n²)` worst-case and `O(n)` best-case. Python's `list.pop(i)` shifts all trailing elements, costing `O(k)` where `k` is the number of elements after `i`. In the worst case (e.g., all elements identical), this results in quadratic time.
- **Space Complexity:** Remains `O(1)` auxiliary, as it modifies the list in-place.
