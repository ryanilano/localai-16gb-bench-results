# 35B_UD-Q4_K_XL — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation using the built-in `unittest` framework:

```python
import unittest

def is_palindrome(s):
    """
    Returns True if `s` is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Filter alphanumeric characters and convert to lowercase
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    # Compare cleaned string with its reverse
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_numbers_and_symbols(self):
        self.assertTrue(is_palindrome("12321!@#"))

    def test_not_a_palindrome(self):
        self.assertFalse(is_palindrome("Hello, World!"))


if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `char.isalnum()` keeps only letters and numbers, and `.lower()` ensures case-insensitivity.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the filtered string with its reverse in O(n) time and space.
3. **Edge Cases Handled**: Empty strings, strings with only punctuation, and single characters all correctly return `True` (standard palindrome definition).

### Run the tests:
Save the code in a file (e.g., `test_palindrome.py`) and run:
```bash
python test_palindrome.py
```
You should see `OK` with 3 passed tests.
