# 35B_UD-Q3_K_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable Python implementation with the function and 3 unit tests using the built-in `unittest` module:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Keep only alphanumeric characters and convert to lowercase
    cleaned = [char.lower() for char in s if char.isalnum()]
    # Check if the filtered sequence reads the same forwards and backwards
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_standard_palindrome(self):
        self.assertTrue(is_palindrome("racecar"))

    def test_palindrome_with_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("hello world!"))


if __name__ == "__main__":
    unittest.main()
```

### Key Details:
- **Case & Character Filtering**: `char.isalnum()` filters out spaces, punctuation, and symbols. `.lower()` ensures case-insensitivity.
- **Palindrome Check**: `cleaned == cleaned[::-1]` efficiently compares the list to its reverse in Python.
- **Time & Space Complexity**: `O(n)` time and `O(n)` space, where `n` is the length of the input string. (For `O(1)` space, a two-pointer approach can be used, but this version prioritizes Pythonic readability.)
- **Edge Cases Handled**: Empty strings and strings with only non-alphanumeric characters correctly return `True` (standard behavior for this problem).

Run the script directly to execute the tests: `python filename.py`
