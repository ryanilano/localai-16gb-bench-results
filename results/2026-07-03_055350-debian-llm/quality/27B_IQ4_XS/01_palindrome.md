# 27B_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a complete, runnable implementation with the function and 3 unit tests using Python's built-in `unittest` framework:

```python
import unittest

def is_palindrome(s):
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = [char.lower() for char in s if char.isalnum()]
    # Compare the cleaned sequence with its reverse
    return cleaned == cleaned[::-1]

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        # Standard palindrome with spaces, punctuation, and mixed case
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_not_palindrome(self):
        # Clear non-palindrome
        self.assertFalse(is_palindrome("race a car"))

    def test_only_non_alphanumeric(self):
        # Edge case: string with only spaces/symbols should be treated as empty (True)
        self.assertTrue(is_palindrome("   ,!@#"))

if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **`char.isalnum()`**: Filters out anything that isn't a letter or number.
2. **`char.lower()`**: Normalizes case so `'A'` and `'a'` are treated identically.
3. **`cleaned == cleaned[::-1]`**: Checks if the filtered list reads the same forwards and backwards.
4. **Unit Tests**: Cover a classic palindrome, a clear failure case, and an edge case with only non-alphanumeric characters.

Run the script directly with `python your_script.py` to execute the tests. All three will pass.
