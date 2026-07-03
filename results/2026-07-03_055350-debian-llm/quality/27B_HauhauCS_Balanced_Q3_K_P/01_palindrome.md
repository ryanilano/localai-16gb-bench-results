# 27B_HauhauCS_Balanced_Q3_K_P — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable implementation with the function and 3 unit tests using Python's built-in `unittest` module:

```python
def is_palindrome(s):
    """Check if a string is a palindrome, ignoring case and non-alphanumeric characters."""
    # Filter out non-alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Compare the cleaned string with its reverse
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        """Test a classic palindrome with mixed case and punctuation."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        """Test a string that is clearly not a palindrome."""
        self.assertFalse(is_palindrome("Hello, World!"))

    def test_numeric_and_special_chars(self):
        """Test a numeric palindrome surrounded by symbols."""
        self.assertTrue(is_palindrome("12321!@#"))


if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **`char.isalnum()`**: Filters out spaces, punctuation, and symbols, keeping only letters and digits.
2. **`char.lower()`**: Normalizes the case so comparisons are case-insensitive.
3. **`cleaned == cleaned[::-1]`**: Efficiently checks if the processed string reads the same forwards and backwards using Python's slice reversal.

### Running the tests:
Save the code in a file (e.g., `palindrome_test.py`) and run it from the terminal:
```bash
python palindrome_test.py
```
You'll see output confirming all 3 tests pass (e.g., `... OK`).
