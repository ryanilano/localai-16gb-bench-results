# 35B_UD-Q3_K_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation along with 3 unit tests using the built-in `unittest` module:

```python
import unittest

def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = ''.join(ch.lower() for ch in s if ch.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_not_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_empty_and_whitespace_only(self):
        self.assertTrue(is_palindrome("   "))


if __name__ == '__main__':
    unittest.main()
```

### 🔍 How it works:
1. **`ch.isalnum()`** filters out spaces, punctuation, and symbols, keeping only letters and numbers.
2. **`.lower()`** ensures case-insensitivity.
3. **`cleaned[::-1]`** reverses the string using Python's slice notation.
4. Comparing the cleaned string to its reverse gives a clean `True`/`False` result.

###
