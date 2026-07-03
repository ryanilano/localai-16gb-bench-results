# 27B_Heretic_NEO_CODE_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here’s a clean implementation with three `unittest`-based tests:

```python
import unittest

def is_palindrome(s: str) -> bool:
    # Keep only alphanumeric characters and normalize to lowercase
    cleaned = ''.join(ch.lower() for ch in s if ch.isalnum())
    # Check if it reads the same forwards and backwards
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):

    def test_palindrome_with_spaces_and_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_not_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))

    def test_edge_cases(self):
        # Empty string is considered a palindrome
        self.assertTrue(is_palindrome(""))
        # String with only non-alphanumeric characters is also a palindrome
        self.assertTrue(is_palindrome("   ,,!! "))


if __name__ == "__main__":
    unittest.main()
```

Run this file with:
- `python -m unittest your_file_name.py`
or:
- `python your_file_name.py`
