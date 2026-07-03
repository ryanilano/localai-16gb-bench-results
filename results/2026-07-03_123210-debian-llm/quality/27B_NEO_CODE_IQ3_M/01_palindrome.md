# 27B_NEO_CODE_IQ3_M — 01_palindrome

## Prompt

Write a Python function is_palindrome(s) that ignores case and non-alphanumeric characters. Include 3 unit tests.

## Response

Here's a clean, Pythonic implementation along with 3 unit tests using Python's built-in `unittest` framework:

```python
def is_palindrome(s: str) -> bool:
    """
    Returns True if the string is a palindrome, ignoring case and non-alphanumeric characters.
    """
    # Keep only alphanumeric characters and convert to lowercase
    cleaned = ''.join(ch.lower() for ch in s if ch.isalnum())
    # Check if the cleaned string reads the same forwards and backwards
    return cleaned == cleaned[::-1]


import unittest

class TestIsPalindrome(unittest.TestCase):
    def test_mixed_case_with_punctuation(self):
        """Test a classic palindrome with spaces, punctuation, and mixed case."""
        self.assertTrue(is_palindrome("A man, a plan, a canal: Panama"))

    def test_non_palindrome(self):
        """Test a string that is not a palindrome."""
        self.assertFalse(is_palindrome("race a car"))

    def test_numbers_and_special_characters(self):
        """Test with numbers and surrounding special characters."""
        self.assertTrue(is_palindrome("12321!@#"))

if __name__ == '__main__':
    unittest.main()
```

### How it works:
1. **Filtering & Normalization**: `''.join(ch.lower() for ch in s if ch.isalnum())` iterates through the string, keeps only alphanumeric characters (`isalnum()`), converts them to lowercase, and joins them back into a clean string.
2. **Palindrome Check**: `cleaned == cleaned[::-1]` compares the cleaned string to its reverse. Python's slice `[::-1]` efficiently creates the reversed version.
3. **Time/Space Complexity**: `O(n)` time and `O(n)` space, where `n` is the length of the input string. This is optimal for readability and performance in Python.

### Running the tests:
Save the code in a file (e.g., `palindrome.py`) and run:
```bash
python palindrome.py
```
You should see output confirming all 3 tests passed.
