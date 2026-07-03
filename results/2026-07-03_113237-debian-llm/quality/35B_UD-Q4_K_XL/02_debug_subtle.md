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

### Exact Bug
The `i += 1` statement executes unconditionally after a `nums.pop(i)`.

### Why It Fails
When `pop(i)` removes an element, all elements to its right shift left by one index. The element that was originally at `i + 1` now occupies index `i`. Because `i` is incremented immediately after the removal, the loop jumps to `i + 1` and **skips** the newly shifted element. That element is never compared to `nums[i - 1]`, so consecutive duplicates (3+ identical values in a row) are not fully removed.

### Minimal Triggering Input
`[1, 1, 1]`
- **Expected:** `1` (list becomes `[1]`)
- **Actual:** `2` (loop removes the middle `1`, shifts the last `1` to index `1`, then increments `i` to `2` and exits)

### Corrected Version
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
By moving `i += 1` into an `else` block, `i` only advances when no removal occurs. If a removal happens, the loop stays at the same `i` to check the newly shifted element against its new predecessor.

### Complexity Impact
**No change.** The algorithm remains `O(n²)` in the worst case. `list.pop(i)` requires shifting all elements to the right of index `i`, which takes `O(k)` time where `k` is the number of elements after `i`. The fix only changes control flow, not the underlying list mutation strategy, so the asymptotic complexity stays identical.
