# 27B_Heretic_Youssofal_Q3_K_M — 02_debug_subtle

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

### Bug Identification
The bug is located in the unconditional increment of the index `i` (`i += 1`). When the code detects a duplicate and removes it using `nums.pop(i)`, the list shifts to the left. The element that was at `i + 1` moves into position `i`. However, the code immediately increments `i`, causing it to skip checking the element that just shifted into the current position. This results in the failure to remove chains of three or more adjacent duplicates.

### Why it Fails
In Python, `list.pop(i)` removes the element at index `i` and shifts all subsequent elements to the left to fill the gap. If `nums[i]` is removed, the new `nums[i]` is the element that was previously at `nums[i+1]`. The algorithm must re-evaluate this new `nums[i]` against `nums[i-1]`. By unconditionally executing `i += 1` after a deletion, the algorithm advances past the newly shifted element, leaving remaining duplicates in the list.

### Minimal Input Triggering Wrong Result
```python
[1, 1, 1]
```
**Behavior:** The code incorrectly returns `[1, 1]` (count 2).
**Expected:** The code should return `[1]` (count 1).

### Corrected Version
The fix ensures that the index `i` is only incremented when no deletion occurs. If a deletion happens, the loop repeats the check at the current index `i` against the new element that has shifted there.

```python
def dedupe_adjacent(nums):
    i = 0
    while i < len(nums):
        if i > 0 and nums[i] == nums[i - 1]:
            nums.pop(i)
            # Do not increment i; check the new element at this index
        else:
            i += 1
    return len(nums)
```

### Complexity Analysis
**Does the fix change the O() complexity?**
No. The original code and the corrected code both rely on `list.pop(i)`, which is an $O(N)$ operation because it requires shifting memory. Consequently, both implementations have a time complexity of **$O(N^2)$** in the worst case. The fix corrects the logic but does not alter the asymptotic complexity class.
