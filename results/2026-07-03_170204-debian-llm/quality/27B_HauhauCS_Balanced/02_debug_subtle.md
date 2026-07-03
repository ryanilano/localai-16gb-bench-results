# 27B_HauhauCS_Balanced — 02_debug_subtle

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

> ⚠️ reasoning only — no final answer (raise GEN). Thinking phase:

Here's a thinking process that leads to the suggested answer:

1.  **Analyze the Request:**
    *   **Goal:** Identify a bug in a provided Python function `dedupe_adjacent`, explain it, provide a failing input, and fix it.
    *   **Constraint:** Do not change the algorithm (keep the logic of iterating and popping in-place).
    *   **Constraint:** State if the fix changes the Big-O complexity.

2.  **Analyze the Code:**
    ```python
    def dedupe_adjacent(nums):
        i = 0
        while i < len(nums):
            if i > 0 and nums[i] == nums[i - 1]:
                nums.pop(i)
            i += 1
        return len(nums)
    ```
    *   *Logic:* It iterates through the list with index `i`. If the current element `nums[i]` matches the previous one `nums[i-1]`, it removes `nums[i]`. Then it increments `i` by 1.
    *   *Mechanism:* `pop(i)` removes the element at index `i` and shifts all subsequent elements to the left (indices decrease by 1).

3.  **Trace the Execution (Mental Walkthrough):**
    *   Let's try a simple case: `[1, 2, 2, 3]`.
        *   `i=0`: `nums[0]` (1). `i>0` is False. `i` becomes 1.
        *   `i=1`: `nums[1]` (2). `nums[0]` (1). Not equal. `i` becomes 2.
        *   `i=2`: `nums[2]` (2). `nums[1]` (2). Equal! `pop(2)` happens. List becomes `[1, 2, 3]`.
        *   `i` increments to 3.
        *   `i=3`: `len(nums)` is 3. Loop terminates.
        *   Result: `[1, 2, 3]`. Correct.

    *   Let's try a triple duplicate case: `[1, 2, 2, 2, 3]`.
        *   `i=0`: Skip. `i` -> 1.
        *   `i=1`: `nums[1]`(2) != `nums[0]`(1). `i` -> 2.
        *   `i=2`: `nums[2]`(2) == `nums[1]`(2). `pop(2)`. List becomes `[1, 2, 2, 3]`.
            *   *Crucial Step:* The element that was at index 3 (the second `2`) shifts to index 2.
        *   `i` increments to 3.
        *   `i=3`: `nums[3]` is now `3`. `nums[2]` is `2`. Not equal. `i` -> 4.
        *   Loop ends.
        *   Result: `[1, 2, 2, 3]`. **Incorrect.** It missed the second `2` because it moved into the spot the loop just checked, but the loop moved on to the next index.

    *   Let's try a crash case (Index out of bounds?):
        *   The condition `i < len(nums)` is checked every iteration.
        *   Inside the loop, `nums.pop(i)` reduces length.
        *   If `i` is at the very end, say `[1, 2]`.
        *   `i=0`: Skip. `i` -> 1.
        *   `i=1`: `nums[1]`(2) != `nums[0]`(1). `i` -> 2.
        *   Loop check `2 < 2` fails. Safe.
        *   What if we pop at the last index? `[1, 1]`.
        *   `i=0`: Skip. `i` -> 1.
        *   `i=1`: `nums[1]`(1) == `nums[0]`(1). `pop(1)`. List is `[1]`. Length is 1.
        *   `i` increments to 2.
        *   Loop check `2 < 1` fails. Safe.
        *   *Self-Correction:* It doesn't seem to crash based on index out of bounds easily because `i` increments *after* the check, and the loop condition checks `len`. However, if `i` increments past the new length, it just stops.

4.  **Identify the Exact Bug:**
    The bug is the `i += 1` increment happening unconditionally after a `pop`.
    When `nums.pop(i)` is executed, the element at `i` is removed, and the element at `i+1` shifts down to `i`. The next element to be checked against `nums[i-1]` (or `nums[i]` for the next step) is now at index `i`. However, the code increments `i` to `i+1`, effectively skipping the element that just shifted into position `i`.

5.  **Formulate the Explanation:**
    *   **The Bug:** Incrementing `i` after removing an element.
    *   **Why it fails:** `pop(i)` shifts subsequent elements to the left. The element originally at `i+1` is now at `i`. By incrementing `i`, the loop moves to `i+1` (which is now the element originally at `i+2`), skipping the check for the element that just moved into `i`.
    *   **Minimal Input:** `[1, 2, 2, 2]` or `[1, 1, 1]`. Let's trace `[1, 1, 1]`.
        *   `i=0`: Skip. `i` -> 1.
        *   `i=1`: `nums[1]`(1) == `nums[0]`(1). Pop index 1. List: `[1, 1]`.
        *   `i` -> 2.
        *   `i=2`: Loop condition `2 < 2` False.
        *   Result `[1, 1]`. Expected `[1]`.
    *   Wait, is there a crash? The prompt mentions "skip elements or crash".
    *   Let's check the crash scenario again.
    *   If `nums` is empty? `len` is 0. Loop doesn't run. Returns 0. OK.
    *   If `nums` has 1 element? Loop runs `i=0`. `i>0` False. `i`->1. Ends. OK.
    *   Maybe the crash happens if `i` goes out of bounds *before* the check? No, `while` checks first.
    *   What if `pop` changes the length such that `i` is valid, but accessing `nums[i]` inside the `if` (though `i` is used in `pop` and comparison) is fine?
    *   Actually, strictly speaking, `pop(i)` is safe as long as `0 <= i < len`. Since the loop condition is `i < len`, it's safe.
    *   However, consider the logic `nums[i] == nums[i-1]`. If `i` is incremented *before* the check, it might be an issue, but here it's at the end.
    *   Is there a crash? Maybe if `nums` is modified such that `i` is valid but `i-1` is out of bounds? No, `i` starts at 0, increments to 1. `i-1` is 0.
    *   Maybe the prompt implies a crash on *specific* inputs or just "potential" crash? Or maybe I missed a subtle crash.
    *   Let's look at `nums.pop(i)`. If `i` equals `len(nums)`? No, `while` prevents that.
    *   Maybe the crash is related to `nums[i-1]`? If `i` becomes 0? No, `i` only increases.
    *   Okay, the "skip elements" part is the definite bug. The "crash" might be a red herring or referring to a scenario where `i` logic is slightly different, but with this specific code, skipping is the primary failure mode.
    *   *Correction:* Wait, if `i` is incremented, and `pop` happened, `i` might point to an index that is now out of bounds?
        *   List `[1, 1]`. `i=1`. `pop(1)`. List `[1]`. `len` is 1. `i` becomes 2. `while 2 < 1` is False. No crash.
    *   Okay, sticking to the "skipping" bug is the most solid answer.

6.  **Develop the Fix:**
    *   We need to stay at the current index `i` if we remove an element, because a new element has shifted into that spot.
    *   **Fix:** Only increment `i` if we *didn't* pop.
    *   Alternatively, use an `else` clause for the `if`.

    ```python
    def dedupe_adjacent(nums):
        i = 0
        while i < len(nums):
            if i > 0 and nums[i] == nums[i - 1]:
                nums.pop(i)
                # Do not increment i here, stay at i to check the new element
            else:
                i += 1
        return len(nums)
    ```

7.  **Analyze Complexity:**
    *   **Original:** `pop(i)` takes O(N) because it shifts elements. The loop runs N times. Total O(N^2).
    *   **Fixed:** `pop(i)` still takes O(N). The loop might run slightly fewer times (or same number), but the shift cost remains. Total O(N^2).
    *   **Conclusion:** Complexity remains O(N^2).

8.  **Refining the "Crash" aspect:**
    *   Is there *any* input that crashes?
    *   If `nums` is a list, `pop` is safe.
    *   What if `nums` is not a list? The prompt implies it's a list.
    *   Maybe the prompt considers `IndexError` if `i` logic was slightly different?
    *   Let's re-read carefully: "skip elements or crash".
    *   If I have `[1, 2, 2]`.
    *   `i=0` (1). `i`->1.
    *   `i=1` (2). `i`->2.
    *   `i=2` (2). `nums[2]==nums[1]`. Pop 2. List `[1, 2]`. `i`->3.
    *   Loop ends. Result `[1, 2]`. Correct.
    *   If I have `[1, 2, 2, 2]`.
    *   `i=0` (1). `i`->1.
    *   `i=1` (2). `i`->2.
    *   `i=2` (2). Pop 2. List `[1, 2, 2]`. `i`->3.
    *   `i=3`. `len` is 3. Loop ends. Result `[1, 2, 2]`. Missed one.
    *   What if `nums` is `[1, 1, 1]`?
    *   `i=0` (1). `i`->1.
    *   `i=1` (1). Pop 1. List `[1, 1]`. `i`->2.
    *   `i=2`. `len` is 2. Loop ends. Result `[1, 1]`. Missed one.
    *   Is there a crash?
    *   Maybe if `i` is used for access *before* checking length? No, `while` checks length.
    *   Maybe if `nums` is empty? `while 0 < 0` -> False. Returns 0.
    *   Maybe the "crash" refers to an infinite loop? No, `i` always increases or list shrinks.
    *   Wait, if `pop` happens, list shrinks. If `i` is not incremented, `i` stays same. `i < len` might still be true.
    *   In the fixed version:
        *   `[1, 1, 1]`.
        *   `i=0`. `i`->1.
        *   `i=1`. Pop. List `[1, 1]`. `i` stays 1.
        *   `i=1`. `nums[1]`(1) == `nums[0]`(1). Pop. List `[1]`. `i` stays 1.
        *   `i=1`. `len` is 1. `1 < 1` False. Loop ends.
        *   Result `[1]`. Correct.
    *   Okay, so the original code definitely has the skipping bug. The crash part might be a generic warning in the prompt or I'm missing a very edge case, but the skipping is the definitive logic error.
    *   *Self-Correction on Crash:* If `i` is incremented to `len(nums)` (which is now smaller due to pop), the loop terminates. No crash.
    *   However, if `i` was decremented or logic was different...
    *   Actually, let's look at `nums[i]` access.
    *   If `i` is valid at start of loop, `nums[i]` is safe.
    *   If `pop(i)` happens, `i` is still valid for the *next* iteration check?
    *   If `pop(i)` happens, `len` decreases by 1.
    *   If `i` was `len-1` (last element), `pop(i)` makes `len` equal to old `i`.
    *   Then `i` increments. `i` becomes `len` (old `i` + 1).
    *   Next loop check `i < len` fails.
    *   So `nums[i]` is never accessed with an out of bounds index.
    *   I will focus on the "skipping" bug as the primary failure, as it is guaranteed to happen on triple duplicates.

9.  **Drafting the Response:**

    *   **Bug Identification:** The index `i` is incremented even when an element is removed (`pop`).
    *   **Explanation:** When `nums.pop(i)` is called, the element at `i` is removed, and the element at `i+1` shifts down to index `i`. By immediately executing `i += 1`, the loop skips over the element that just shifted into position `i`, failing to compare it with the previous element.
    *   **Minimal Input:** `[1, 2, 2, 2]` (or `[1, 1, 1]`).
        *   Trace for `[1, 2, 2, 2]`:
            1. `i=0`, val=1. `i` becomes 1.
            2. `i=1`, val=2. Prev=1. No match. `i` becomes 2.
            3. `i=2`, val=2. Prev=2. Match. `pop(2)`. List is `[1, 2, 2]`.
            4. `i` becomes 3.
            5. `i=3`. `len` is 3. Loop ends.
            6. Result `[1, 2, 2]`. Expected `[1, 2]`.
    *   **Fix:** Move `i += 1` into an `else` block.
    *   **Complexity:** `pop` is O(N). Loop is O(N). Total O(N^2). Fix does not change this.

10. **Refining the "Crash" part (Just to be thorough):**
    Is it possible the prompt implies a crash?
    If `nums` is a list of references? No.
    Maybe if `i` is not checked properly?
    Actually, if `nums` is `[1, 2, 2]` and we are at `i=2`. `pop(2)`. List `[1, 2]`. `i` becomes 3. Loop ends.
    If `nums` is `[1, 1]`. `i=1`. `pop(1)`. List `[1]`. `i` becomes 2. Loop ends.
    It seems strictly a skipping bug. I will mention that it causes incorrect results (skipping), and perhaps "crash" in the prompt is a general descriptor for "failure" or referring to a specific edge case I'm not seeing (maybe `IndexError` if `i` logic was different, but here it's safe).
    *Wait*, if `nums` is empty? `len` 0. `while` false. Returns 0.
    If `nums` is `[1]`. `i=0`. `i>0` false. `i`->1. Loop ends. Returns 1.
    Okay, the skipping is the main issue.

    *Self-Correction Check:* Is there any scenario where `nums[i-1]` crashes?
    `i` starts at 0. `i > 0` check protects `nums[i-1]`.
    `i` increments.
    So `i-1` is always >= 0 when accessed.
    So no crash on `i-1`.
    `nums[i]` is accessed inside `if` (implicitly, via `pop` or comparison).
    `while i < len(nums)` protects `nums[i]`.
    So, strictly speaking, the code *doesn't* crash on standard lists. It just produces wrong results. However, I will acknowledge the prompt's wording "skip elements or crash" by focusing heavily on the skip, as that is the definitive bug.

    *Wait, could it crash if `nums` is not a list but an iterator?* The prompt says "list".
    *Could it crash if `nums` contains unhashable types?* No, just equality check.

    Let's stick to the logic error.

11. **Final Polish of the Answer:**
    *   **Bug:** Incrementing `i` after `pop`.
    *   **Why:** Shift happens, index moves past the new element at `i`.
    *   **Input:** `[1, 2, 2, 2]` or `[1, 1, 1]`.
    *   **Fix:** `else: i += 1`.
    *   **Complexity:** Unchanged (O(N^2)).

    Let's double check the "crash" possibility one last time.
    If `nums` is `[1, 2, 2]`.
    `i=2`. `pop(2)`. List `[1, 2]`. `i` becomes 3.
    Next iteration check `3 < 2`. False.
    What if `nums` is `[1, 2, 2, 3]`.
    `i=2` (val 2). Pop. List `[1, 2, 3]`. `i` becomes 3.
    `i=3` (val 3). `nums[3]` vs `nums[2]`. `3` vs `2`. OK.
    Wait, `nums` is `[1, 2, 3]`. Indices 0, 1, 2.
    `i` is 3. `len` is 3.
    Loop terminates.
    It seems robust against crashes. The "crash" in the prompt might be a slight exaggeration or referring to the fact that logically the state is broken (like a runtime error in logic), or perhaps if `nums` was a specific data structure where `pop` behaves differently? No, standard list.
    Actually, maybe if `i` is incremented *before* the check? No, code is clear.
    I'll stick to the skipping explanation.

    *One edge case:* What if `nums` is modified by an external observer? Not applicable.
    What if `nums` is `[1, 1, 1, 1]`?
    `i=0` -> 1.
    `i=1` (1). Pop. List `[1, 1, 1]`. `i` -> 2.
    `i=2` (1). `nums[2]` vs `nums[1]`. `1` vs `1`. Pop. List `[1, 1]`. `i` -> 3.
    `i=3`. `len` 2. Stop.
    Result `[1, 1]`.
    It seems it skips *every other* duplicate in a long run.

    **Final Answer Structure:**
    1.  Identify Bug.
    2.  Explain Why.
    3.  Minimal Input.
    4.  Corrected Code.
    5.  Complexity Note.

    **Corrected Code Snippet:**
    ```python
    def dedupe_adjacent(nums):
        i = 0
        while i < len(nums):
            if i > 0 and nums[i] == nums[i - 1]:
                nums.pop(i)
                # Bug fix: Do not increment i here
            else:
                i += 1
        return len(nums)
    ```
    Or simply `i += 1` inside `else`.

    **Complexity Check:**
    `pop(i)` shifts $O(N)$ elements.
    In worst case (all distinct), loop runs $N$ times, no pops. $O(N)$.
    In worst case (all same), loop runs $N$ times, pops $N-1$ times. Each pop is $O(N)$ (decreasing). Sum $1+2+...+N \approx O(N^2)$.
    So complexity is $O(N^2)$.
    The fix doesn't change the operations performed, just the flow control. Complexity remains $O(N^2)$.

    Wait, is there a crash on `[1, 1]`?
    `i=0`. `i`->1.
    `i=1`. `nums[1]` (1) == `nums[0]` (1). `pop(1)`. List `[1]`.
    `i` -> 2.
    `while 2 < 1` -> False.
    No crash.

    Is there a crash on `[1]`?
    `i=0`. `i`->1.
    `while 1 < 1` -> False.
    No crash.

    Is there a crash on `[]`?
    `while 0 < 0` -> False.
    No crash.

    Maybe the crash refers to `IndexError` if `i` was not checked against `len` before access, but it is.
    I will assume "crash" in the prompt is a generic term for "fail" or perhaps they consider returning a wrong length a "logical crash". Or maybe I am blind to a specific case.
    *Hypothesis:* If `nums` is a list of lists? No.
    *Hypothesis:* If `nums` is a generator? No, `len` would fail.
    *Hypothesis:* Maybe `nums[i-1]` access? If `i` becomes 0? `i` starts 0, increments. `i` is never 0 inside the `if` because of `i > 0` check.
    Okay, I'll stick to the logic error.

    **Wait, looking at the code again:**
    `if i > 0 and nums[i] == nums[i - 1]:`
    If `i` is 0, `nums[i-1]` is not evaluated (short-circuit).
    If `i` > 0, `nums[i-1]` is safe.
    `nums[i]` is safe because of `while`.
    `pop(i)` is safe.
    `i += 1`.
    It seems perfectly safe from `IndexError`.

    **Minimal Input for Wrong Result:**
    `[1, 2, 2, 2]` -> `[1, 2, 2]` (Expected `[1, 2]`).
    `[1, 1, 1]` -> `[1, 1]` (Expected `[1]`).
    `[1, 1, 1]` is shorter, so it's a better "minimal" input.

    **Final check of the fix:**
    If I use `else: i += 1`, does it handle `[1, 2, 2, 2]`?
    `i=0` (1). `else` -> `i=1`.
    `i=1` (2). `else` -> `i=2`.
    `i=2` (2). Match `nums[1]`. `pop(2)`. List `[1, 2, 2]`. No else (i stays 2).
    `i=2` (2). Match `nums[1]`. `pop(2)`. List `[1, 2]`. No else (i stays 2).
    `i=2`. `len` is 2. Loop ends.
    Result `[1, 2]`. Correct.

    Does it handle `[1, 2, 3]`?
    `i=0` -> 1.
    `i=1` (2) != 1 -> 2.
    `i=2` (3) != 2 -> 3.
    End. Correct.

    Does it handle `[1, 1]`?
    `i=0` -> 1.
    `i=1` (1) == 1. Pop. List `[1]`. `i` stays 1.
    `i=1`. `len` 1. Loop ends.
    Result `[1]`. Correct.

    The fix works.

    **Complexity:**
    The fix keeps the loop structure. `pop` is still $O(N)$.
    Complexity is unchanged.

    **One detail:**
    The prompt asks "Identify the exact bug".
    Bug: `i` is incremented after `pop`, skipping the element that shifted into index `i`.

    **Explanation:**
    When `nums.pop(i)` is executed, the list shrinks and elements to the right shift left. The element that was at `i+1` moves to `i`. The code then executes `i += 1`, moving the index to `i+1` (which is now the element originally at `i+2`), effectively ignoring the new element at `i`.

    **Minimal Input:**
    `[1, 1, 1]` (or `[1, 2, 2, 2]`). `[1, 1, 1]` is minimal length 3.

    **Corrected Version:**
    Use `else` for increment.

    **Complexity:**
    O(N^2) remains.

    Ready to write.

    Wait, is there *any* crash?
    Maybe if `nums` is `[1, 2, 2]` and `i` is 2.
    `pop(2)`. List `[1, 2]`.
    `i` becomes 3.
    Loop check `3 < 2` -> False.
    It works.
    Maybe the prompt implies a crash if `nums` is modified in a way that `len` changes unexpectedly? No.
    Maybe the prompt implies a crash if `i` is not an integer? No.
    Maybe the prompt implies a crash if `nums` is a tuple? `pop` would raise AttributeError. But prompt says "list".
    I will ignore the "crash" part as a generic descriptor or potential edge case not strictly present in standard list behavior, but focus on the skipping which is the fatal logic flaw.
    *Self-Correction:* Actually, if `nums` is `[1, 2, 2]`. `i` starts 0.
    1. `i=0`. `i`->1.
    2. `i=1`. `nums[1]`(2) != `nums[0]`(1). `i`->2.
    3. `i=2`. `nums[2]`(2) == `nums[1]`(2). `pop(2)`. List `[1, 2]`. `i`->3.
    4. `i=3`. `len` 2. Stop.
    Wait, in step 3, `nums` was `[1, 2, 2]`. `nums[2]` is 2. `nums[1]` is 2.
    Wait, `nums[1]` is 2?
    Original list `[1, 2, 2]`.
    Index 0: 1.
    Index 1: 2.
    Index 2: 2.
    Yes.
    So it works for `[1, 2, 2]`.
    It fails for `[1, 2, 2, 2]`.
    Step 1: `i=0`. `i`->1.
    Step 2: `i=1`. `nums[1]`(2) != `nums[0]`(1). `i`->2.
    Step 3: `i=2`. `nums[2]`(2) == `nums[1]`(2). `pop(2)`. List `[1, 2, 2]`. `i`->3.
    Step 4: `i=3`. `len` 3. Stop.
    Result `[1, 2, 2]`.
    Wait, original `[1, 2, 2, 2]`.
    Indices: 0:1, 1:2, 2:2, 3:2.
    `i=2`. `nums[2]` is 2. `nums[1]` is 2. Match.
    Pop index 2.
    List becomes `[1, 2, 2]`. (The element at old index 3 shifted to index 2).
    `i` becomes 3.
    Loop checks `3 < 3`. False.
    Result `[1, 2, 2]`.
    Wait, the element at new index 2 (which was old index 3) is 2.
    It was never compared to `nums[1]` (which is 2).
    So yes, it skipped.

    What about `[1, 1, 1]`?
    Indices: 0:1, 1:1, 2:1.
    `i=0`. `i`->1.
    `i=1`. `nums[1]`(1) == `nums[0]`(1). Pop 1. List `[1, 1]`. `i`->2.
    `i=2`. `len` 2. Stop.
    Result `[1, 1]`.
    Skipped the check for the element that moved to index 1.

    Okay, explanation is solid.

    **Final check on "Crash"**:
    Is it possible `nums.pop(i)` raises IndexError?
    Only if `i >= len`.
    Loop condition `i < len` ensures `i` is valid index.
    Unless `len` changes between check and pop? No, single threaded.
    So no crash.

    Maybe the prompt meant "crash" as in "infinite loop"?
    If `i` was decremented? No.
    If `pop` didn't reduce length? (Custom object).
    But for standard list, no crash.
    I'll stick to "skipping elements".

    **Wait, what if the input is `[1, 2, 2, 3, 3, 3]`?**
    `i` goes 0->1->2.
    At 2 (val 2), pop. List `[1, 2, 3, 3, 3]`. `i`->3.
    At 3 (val 3). Compare `nums[3]`(3) vs `nums[2]`(2). No match. `i`->4.
    At 4 (val 3). Compare `nums[4]`(3) vs `nums[3]`(3). Match. Pop. List `[1, 2, 3, 3]`. `i`->5.
    At 5. `len` 4. Stop.
    Result `[1, 2, 3, 3]`.
    It missed the second 3 (which shifted to index 3).
    It works correctly for the first group of duplicates if they are at the end of the list?
    No, `[1, 2, 2, 2]` failed.
    Wait, in `[1, 2, 2, 3, 3, 3]`, the first group `2, 2` was handled?
    Let's re-trace `[1, 2, 2, 2]`.
    `i=0` (1). `i`->1.
    `i=1` (2). `i`->2.
    `i=2` (2). Match `nums[1]`. Pop. List `[1, 2, 2]`. `i`->3.
    Loop ends.
    It missed the last 2.
    So yes, it fails for any group of 3+ duplicates, or multiple groups where the second group is adjacent to a popped element?
    Actually, it fails for *any* duplicate sequence of length >= 3.
    And it fails for `[1, 2, 2, 3, 3]`?
    `i=0`->1.
    `i=1` (2) -> 2.
    `i=2` (2) -> pop. List `[1, 2, 3, 3]`. `i`->3.
    `i=3` (3). Compare `nums[3]`(3) vs `nums[2]`(2). No match. `i`->4.
    `i=4` (3). Compare `nums[4]`(3) vs `nums[3]`(3). Match. Pop. List `[1, 2, 3]`. `i`->5.
    Stop.
    Result `[1, 2, 3]`.
    Wait, `[1, 2, 2, 3, 3]` -> `[1, 2, 3]`. That looks correct?
    Original: 1, 2, 2, 3, 3.
    Duplicates removed: one 2, one 3.
    Result 1, 2, 3.
    Why did it work here?
    Because after popping the 2 at index 2, the 3 shifted to index 2.
    `i` became 3.
    So `i` skipped index 2.
    Index 2 now holds 3.
    `i` is 3. `nums[3]` is 3 (the second 3).
    It compared `nums[3]` (3) with `nums[2]` (3).
    Wait, `nums[2]` is 3 (the one that shifted). `nums[3]` is 3 (the original second 3).
    So it found a match!
    Wait, trace carefully.
    List: `[1, 2, 2, 3, 3]`. Indices 0,1,2,3,4.
    `i=0` (1). `i`->1.
    `i=1` (2). `i`->2.
    `i=2` (2). Match `nums[1]`(2).
