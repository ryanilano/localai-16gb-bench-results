# 35B_UD-Q4_K_XL — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation along with 3 unit tests using the built-in `unittest` framework:

```python
def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter only alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_numbers_and_symbols(self):
        self.assertTrue(is_palindrome("12321"))

if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `''.join(char.lower() for char in s if char.isalnum())` iterates through the string, keeps only letters/numbers, converts them to lowercase, and joins them back into a single string.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the cleaned string with its reverse. Slicing with `[::-1]` efficiently creates a reversed copy.
3. **Complexity**: 
   - ⏱️ Time: `O(n)` where `n` is the length of the input string
   - 💾 Space: `O(n)` for the cleaned string (can be optimized to `O(1)` space with a two-pointer approach if memory is a strict constraint)

### To run the tests:
Save the code in a file (e.g., `test_palindrome.py`) and run:
```bash
python test_palindrome.py
```
You should see `OK` indicating all 3 tests passed.
