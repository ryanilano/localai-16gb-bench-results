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
    cleaned = ''.join(char.lower() for char in s if char.isalnum())
    return cleaned == cleaned[::-1]


class TestIsPalindrome(unittest.TestCase):
    def test_classic_palindrome_with_punctuation(self):
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))
        
    def test_not_a_palindrome(self):
        self.assertFalse(is_palindrome("race a car"))
        
    def test_empty_or_only_symbols(self):
        # After removing non-alphanumeric characters, it becomes an empty string,
        # which is considered a valid palindrome.
        self.assertTrue(is_palindrome("   ,,, "))


if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **Filtering**: `char.isalnum()` keeps only letters and numbers.
2. **Case normalization**: `.lower()` ensures case-insensitive comparison.
3. **Reversal check**: `cleaned == cleaned[::-1]` compares the string to its reverse using Python's slice notation.

### Running the tests:
Save the code in a file (e.g., `test_palindrome.py`) and run:
```bash
python test_palindrome.py
```
You'll see:
```
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```
