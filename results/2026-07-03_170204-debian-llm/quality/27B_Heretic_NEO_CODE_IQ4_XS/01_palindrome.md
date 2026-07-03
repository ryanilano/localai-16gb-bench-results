# 27B_Heretic_NEO_CODE_IQ4_XS — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here’s a clean implementation with three unit tests using Python’s `unittest` module:

```python
def is_palindrome(s: str) -> bool:
    # Keep only alphanumeric characters and normalize to lowercase
    cleaned = ''.join(ch.lower() for ch in s if ch.isalnum())
    # Check if it reads the same forwards and backwards
    return cleaned == cleaned[::-1]


if __name__ == "__main__":
    import unittest

    class TestIsPalindrome(unittest.TestCase):
        def test_mixed_case_with_punctuation(self):
            self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

        def test_not_palindrome(self):
            self.assertFalse(is_palindrome("hello"))

        def test_numeric_with_spaces(self):
            self.assertTrue(is_palindrome("12321"))

    unittest.main()
```

To run the tests, save this code in a file (e.g., `palindrome_test.py`) and execute:
- `python palindrome_test.py`
