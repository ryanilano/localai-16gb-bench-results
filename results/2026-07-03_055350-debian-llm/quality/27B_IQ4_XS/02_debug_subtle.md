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

### 🔍 Exact Bug
The unconditional `i += 1` at the end of the loop. When `nums.pop(i)` removes an element, all subsequent elements shift left by one index. The next element to evaluate now sits at the **current** index `i`, but incrementing `i` causes the algorithm to skip it entirely.

### 📉 Why It Fails
After a `pop(i)`, the list shrinks and `nums[i]` now holds the element that was previously at `i+1`. By always incrementing `i`, the loop never compares this newly shifted element against its predecessor. This causes:
1. **Missed duplicates**: Sequences of 3+ identical values won't be fully reduced.
2. **Incorrect final count**: The function returns a length larger than the true deduplicated length.

*(Note: The `while i < len(nums)` guard prevents an actual `IndexError` crash, but the logical skip produces silently wrong results.)*

### 🧪 Minimal Failing Input
```python
dedupe_adjacent([1, 1, 1])
```
- **Expected**: `1` (list becomes `[1]`)
- **Actual**: `2` (list becomes `[1, 1]`)

**Trace of failure**:
1. `i=0`: condition false → `i` becomes 1
2. `i=1`: `nums[1] == nums[0]` → `pop(1)` → `nums = [1, 1]` → `i` becomes 2
3. `while 2 < 2` → loop exits prematurely. Second duplicate is never checked.

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
*Change*: Only increment `i` when no element is removed. If a `pop` occurs, `i` stays the same so the next iteration correctly evaluates the newly shifted element.

### ⏱️ Complexity Impact
**No change.** The fix only alters control flow. The asymptotic time complexity remains **O(N²)** because `list.pop(i)` requires shifting all trailing elements left, which is an O(N) operation. The loop still runs at most N times. Space complexity remains O(1) as requested.
