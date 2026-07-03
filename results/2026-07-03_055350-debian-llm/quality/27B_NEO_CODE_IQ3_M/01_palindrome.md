# 27B_NEO_CODE_IQ3_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's the complete implementation with the function and three unit tests using Python's built-in `unittest` framework:

```python
def is_palindrome(s):
    # Filter alphanumeric characters and convert to lowercase
    cleaned = "".join(c.lower() for c in s if c.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_with_punctuation(self):
        """Test a classic palindrome with spaces, punctuation, and mixed case."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        """Test a regular string that should not be a palindrome."""
        self.assertFalse(is_palindrome("Hello, World!"))

    def test_only_special_characters(self):
        """Test a string containing only non-alphanumeric characters (treated as empty/palindrome)."""
        self.assertTrue(is_palindrome("!@#$%^&*()_+"))

if __name__ == "__main__":
    unittest.main()
```

### How it works:
1. **`c.isalnum()`** filters out spaces, punctuation, and symbols.
2. **`c.lower()`** ensures case-insensitivity.
3. **`cleaned == cleaned[::-1]`** efficiently checks if the processed string is a palindrome by comparing it to its reverse.
4. The `unittest` class contains 3 distinct test cases covering: valid palindromes with noise, non-palindromes, and edge cases with only special characters.

### Run the tests:
Save the code in a file (e.g., `palindrome.py`) and run:
```bash
python palindrome.py
```
Or run it directly as a module:
```bash
python -m unittest palindrome.py
```
