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

### 🔍 Exact Bug
The unconditional `i += 1` executes even when `nums.pop(i)` removes an element. This causes the loop to **skip checking the element that shifts into index `i`** after the removal.

### 📉 Why It Fails
When `pop(i)` removes an element, all subsequent elements shift left by one position. The element originally at `i+1` now occupies index `i`. Because `i` is incremented regardless, the next iteration compares `nums[i+1]` (the original `i+2`) with `nums[i]` (the original `i+1`), completely bypassing the newly shifted element. This leaves extra adjacent duplicates in the list and produces an incorrect count.

### 🧪 Minimal Failing Input
```python
dedupe_adjacent([1, 1, 1])
```
**Expected:** `[1]` → returns `1`  
**Actual:** `[1, 1]` → returns `2`  
*(The second `1` shifts into index 1 after the first pop, but `i` increments to 2, skipping the comparison.)*

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
**Change:** Moved `i += 1` into an `else` block so the index only advances when no element is removed. This ensures the newly shifted element is re-evaluated against its left neighbor.

### 📊 Complexity Impact
**No change.** The fix preserves the original algorithm's asymptotic complexity:
- **Time:** `O(N²)` worst-case. `list.pop(i)` requires shifting all elements to the right of `i`, which is `O(k)`. In the worst case (all duplicates), this shifting happens `O(N)` times, yielding `O(N²)`.
- **Space:** `O(1)` auxiliary (modifies in-place).

The correction only changes control flow, not the underlying data structure operations, so the Big-O remains identical.
