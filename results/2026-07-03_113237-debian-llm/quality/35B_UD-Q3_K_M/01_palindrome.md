# 35B_UD-Q3_K_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable Python script with the function and 3 unit tests using the built-in `unittest` module:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = [char.lower() for char in s if char.isalnum()]
    # Check if the cleaned list reads the same forwards and backwards
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome_with_spaces(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_numeric_and_empty_strings(self):
        self.assertTrue(is_palindrome("12321"))
        self.assertTrue(is_palindrome(""))  # Empty string is considered a palindrome


if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `[char.lower() for char in s if char.isalnum()]` removes all non-alphanumeric characters and converts the remaining ones to lowercase.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the filtered list with its reverse.
3. **Complexity**: O(n) time and O(n) space, where `n` is the length of the input string. This is optimal for readability and typical use cases.

### How to run:
Save the code in a file (e.g., `test_palindrome.py`) and run:
```bash
python test_palindrome.py
```
You'll see output like:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```
