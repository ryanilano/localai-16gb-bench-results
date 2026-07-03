# 27B_Heretic_NEO_CODE_IQ3_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here’s a clean implementation with three unit tests using Python’s built-in `unittest` framework:

```python
def is_palindrome(s: str) -> bool:
    """
    Returns True if s is a palindrome when ignoring case and non-alphanumeric characters.
    """
    cleaned = "".join(ch.lower() for ch in s if ch.isalnum())
    return cleaned == cleaned[::-1]


# Unit tests
import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_with_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_simple_palindrome(self):
        self.assertTrue(is_palindrome("racecar"))

    def test_not_a_palindrome(self):
        self.assertFalse(is_palindrome("Hello, World!"))

if __name__ == "__main__":
    unittest.main()
```

To run the tests, save this code in a file (e.g., `test_palindrome.py`) and execute:
```bash
python test_palindrome.py
```
