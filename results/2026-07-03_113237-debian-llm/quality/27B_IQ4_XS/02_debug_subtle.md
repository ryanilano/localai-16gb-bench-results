# 27B_IQ4_XS — 02_debug_subtle

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
The unconditional `i += 1` at the end of the loop causes the algorithm to **skip elements** after a `pop()` operation.

### 📖 Why It Fails
When `nums.pop(i)` is called, all elements to the right of index `i` shift left by one position. The element that was originally at `i+1` now occupies index `i`. However, because `i` is incremented regardless of whether a `pop` occurred, the loop advances past this newly shifted element without comparing it to its new left neighbor. This leaves consecutive duplicates (3 or more) or adjacent duplicates following other elements intact.

*(Note: In Python, this typically produces wrong results rather than crashing due to the `while i < len(nums)` guard, but in strict index-managed loops or other languages, this pattern would quickly cause an `IndexError`.)*

### 🧪 Minimal Failing Input
```python
dedupe_adjacent([1, 1, 1])
# Expected: [1] (returns 1)
# Actual:   [1, 1] (returns 2)
```

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
*Only the control flow is adjusted: `i` is incremented **only when no element is removed**, ensuring the newly shifted element at index `i` is re-evaluated in the next iteration.*

### 📊 Complexity Impact
**No change.** The fix does not alter the asymptotic complexity:
- **Time:** `O(n²)` in the worst case. `list.pop(i)` requires shifting all subsequent elements, which takes `O(k)` time. In the worst case (e.g., all identical elements), this happens `O(n)` times.
- **Space:** `O(1)` auxiliary (modifies the list in-place).

The correction only fixes the index traversal logic; it does not replace the underlying `O(k)` shifting operation.
