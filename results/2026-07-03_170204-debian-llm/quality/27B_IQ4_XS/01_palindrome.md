# 27B_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, production-ready implementation with `unittest`-based tests:

```python
import unittest

def is_palindrome(s: str) -> bool:
    # Filter out non-alphanumeric characters and normalize case
    cleaned = "".join(char.lower() for char in s if char.isalnum())
    # Compare the cleaned string with its reverse
    return cleaned == cleaned[::-1]

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_numeric_palindrome(self):
        self.assertTrue(is_palindrome("12321"))

if __name__ == "__main__":
    unittest.main()
```

### Key Details:
- **`char.isalnum()`** efficiently filters out spaces, punctuation, and symbols.
- **`char.lower()`** ensures case-insensitivity.
- **`cleaned == cleaned[::-1]`** leverages Python's slice notation for a concise palindrome check.
- The tests cover: mixed case/punctuation, a clear non-palindrome, and numeric-only input. Run the script directly with `python script.py` to execute the test suite.
